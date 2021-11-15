<template>
	<div id='container'>
		<div class='title'>
			<a href="https://www.anyrtc.io/" target='_blank'>anyRTC samples</a> 
			<span>Peer connection</span>
		</div>
		<div class='instructions'>The local user id is <span class='userId'>{{ Config.uid }}</span></div>
		<div id='playContainer'>
			<div v-if='isLogin' id='local'></div>
			<div v-if='isLogin'>
				<div v-for='(user, index) in list' :key='index' :id="'distal' + user.uid" class='distal'></div>
			</div>
		</div>
		<button v-if='isLogin' @click='hangUp' class='btn'>Hang Up</button>
		<input v-if='!isLogin' type="text" v-model='channel' class='channelInput' placeholder='Please enter the channel'/>
		<button v-if='!isLogin' @click='join' :disabled='!channel' class='joinBtn'>join</button>
		<div class='instructions'>View the console to see logging. The MediaStream object localStream, and the RTCPeerConnection objects pc1 and pc2 are in global scope, so you can inspect them in the console as well.</div>
		<div class='instructions'>For more information about anyRTC WebRTC, see Getting Started With <a href="https://docs.anyrtc.io/cn/Video/api-ref/rtc_web/overview" target='_blank'>anyRTC</a></div>
		<a href="https://github.com/941725931/webRTC_React" id="viewSource" target='_blank'>View source on GitHub</a>
    </div>
</template>

<script>
import ArRTC from 'ar-rtc-sdk';
import Config from './anyrtc_config';
export default {
    data() {
		return {
			channel: '',
			isLogin: false,
			rtcClient: null,
			videoDevices: null,
			audioDevices: null,
			audioTrack: null,
			videoTrack: null,
			list: [],
			Config
		}
	},
	mounted() {
		this.createRtcClient();
	},
	methods: {
		createRtcClient() {
			const config = { mode: "rtc", codec: "h264" };
			this.rtcClient = ArRTC.createClient(config);
			this.listenUserPublished();
			this.listenUserUnpublished();
		},
		listenUserUnpublished() {
			this.rtcClient.on("user-unpublished", async (user, mediaType) => {
				if (mediaType === 'video') {
					const index = this.list.findIndex(item => item.uid === user.uid);
					if (index !== -1) {
						this.list.splice(index, 1);
					}
				}
			});
		},
		listenUserPublished() {
			this.rtcClient.on("user-published", async (user, mediaType) => {
				await this.rtcClient.subscribe(user, mediaType);
				if (mediaType === 'video') {
					this.list.push(user);
					this.$nextTick(() => {
						user.videoTrack.play('distal' + user.uid);
					});
				} else if (mediaType === 'audio') {
					user.audioTrack.play();
				}
			});
		},
		async getDevices() {
			const [ videoDevices, audioDevices ] = await Promise.all([
				ArRTC.getCameras(),
				ArRTC.getMicrophones(),
			]);
			this.videoDevices = videoDevices;
			this.audioDevices = audioDevices;
		},
		async createTrack() {
			this.audioTrack = await ArRTC.createMicrophoneAudioTrack();
			if (this.videoDevices.length) {
				this.videoTrack = await ArRTC.createCameraVideoTrack();
			}
		},
		join() {
			const { channel } = this;
			const { appid, uid } = Config;
			this.rtcClient.join(appid, channel, null, uid).then(async () => {
				this.isLogin = true; 
				await this.getDevices();
				await this.createTrack();
				this.videoTrack && this.videoTrack.play('local');
				this.audioTrack && this.rtcClient.publish(this.audioTrack);
				this.videoTrack && this.rtcClient.publish(this.videoTrack);
			}).catch(() => {
				alert('加入频道失败');
			});
		},
		hangUp() {
			this.videoTrack && this.videoTrack.stop();
			this.rtcClient.leave();
			Object.assign(this.data, this.$options.data());
		}
	}
}
</script>

<style>
#container {
    margin: 0 auto 0 auto;
    max-width: 60em;
    padding: 1em 1.5em 1.3em 1.5em;
}

.title {
    border-bottom: 1px solid #ccc;
    margin: 0 0 0.8em 0;
    padding: 0 0 0.2em 0;
    font-family: 'Roboto', sans-serif;
    font-weight: 500;
    font-size: 2em;
    white-space: nowrap;
}

.title a, .instructions a {
    text-decoration: none;
    margin-right: 10px;
    color: #1D6EEE;
}

.title a:hover {
    border-bottom: 2px solid #1D6EEE;
}

.instructions {
    margin: 0.8em 0;
    font-family: 'Roboto', sans-serif;
    color: #444;
    font-weight: 300;
}

.instructions .userId {
    color: #1D6EEE;
}

#playContainer {
    display: flex;
    align-items: center;
}

#local, .distal {
    width: 400px;
    height: 220px;
    margin-right: 20px;
    background-color: #222222;
}

.btn {
    width: 70px;
    margin-top: 10px;
    line-height: 26px;
    text-align: center;
    color: #fff;
    background-color: #409EFF;
    border: none;
    border-radius: 2px;
    cursor: pointer;
    transition: .1s;
}

.btn:active {
    transform: translateY(2px);
}

.channelInput {
    width: 240px;
    height: 32px;
    border-radius: 4px;
    border: 1px solid #409EFF;
    outline: none;
    text-indent: .5rem;
    margin-right: 10px;
}

.joinBtn {
    width: 80px;
    line-height: 34px;
    text-align: center;
    color: #fff;
    background-color: #409EFF;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: .1s;
}

.joinBtn:active {
    transform: translateY(2px);
}

.joinBtn[disabled] {
    background-color: #E1E1E2;
    cursor: not-allowed;
}

#viewSource {
    display: block;
    margin: 1.3em 0 0 0;
    border-top: 1px solid #999;
    padding: 1em 0 0 0;
    color: #1D6EEE;
    font-weight: 300;
    text-decoration: none;
}

#viewSource:hover, .instructions a:hover {
    text-decoration: underline;
}
</style>
