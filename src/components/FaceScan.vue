<template>
    <AlertDialog
      ref="alert"></AlertDialog>

    <ConfirmDialog
      ref="confirm"></ConfirmDialog>
    <div class="btn_div">
        <v-btn variant="tonal"
          v-on:click="click_btn" >
            {{ button_name }}
        </v-btn>
    </div>
    <div>
        <table>
            <tr>
                <td>
                    <div id="cameraArea">
                        <video id="id_video"
                            ref="ref_video"
                            :width="video_width"
                            :height="video_height"
                            @loadeddata="onVideoPlay"
                            muted playsinline></video>
                        <canvas id="id_canvas"
                            ref="ref_canvas"
                            class="canvas"
                            :width="video_width"
                            :height="video_height"
                            ></canvas>
                    </div>
                </td>
                <td>
                    <div id="emotionBar" class="emotion" v-bind:style="div_style_emotion">
                        <span id="emotion" class="span_emotion" v-bind:style="style_emotion">{{ txt_emotion }}</span>
                    </div>
                </td>
            </tr>
            <tr>
                <textarea v-model.trim="text_area"></textarea>
            </tr>
        </table>
    </div>
</template>

<script>
const faceapi = require('face-api.js');
import ConfirmDialog from '../components/ConfirmDialog';
import AlertDialog from '../components/AlertDialog';

export default {
    name:'FaceScan',
    components: {
      ConfirmDialog,
      AlertDialog
    },
    data() {
        const button_name_start = "映像を開始";
        const button_name_end = "映像を停止";
        return {
            button_name : button_name_start,
            button_name_start : button_name_start,
            button_name_end : button_name_end,
            MODEL_URL: "/models",
            video_width: 500,
            video_height: 500,
            streamingSrc: null,
            video_start: false,
            videoCanvasCtx: null,
            videocanvas_interval : null,
            emotionTxt : [':)',':|'],
            txt_emotion : "",
            style_emotion : "",
            div_style_emotion: "",
            canvasW : 0,
            canvasH : 0,
            text_area : ""
        }
    },
    mounted() {
        Promise.all([
            //モデルの読み込み
            faceapi.nets.tinyFaceDetector.load(this.MODEL_URL),
            faceapi.nets.faceExpressionNet.load(this.MODEL_URL),
            faceapi.nets.faceLandmark68Net.load(this.MODEL_URL),
            faceapi.nets.faceLandmark68TinyNet.load(this.MODEL_URL),
            faceapi.nets.faceRecognitionNet.load(this.MODEL_URL),
            faceapi.nets.mtcnn.load(this.MODEL_URL)
        ]).then(() => {
           console.log("成功"); 
        }).catch(() => {
            console.log("失敗"); 
        });

        this.streamingSrc = null;
        this.video_start = false;

        this.txt_emotion = this.emotionTxt[1]; 
        
    },
    methods: {
        alert : async function(message) {
            console.log("alert :" + message);
            return await this.$refs.alert.open(message);
        },
        confirm: async function(title, message) {
            console.log("confirm :" + message);
            return await this.$refs.confirm.open(title, message, "OK");
        },
        click_btn : function () {
            console.log("click_btn");
            //this.alert("ボタン プッシュ");

            this.videoStart();
        },
        videoStart: function() {
            console.log("videoStart");
            if (this.streamingSrc != null) {
                this.streamingSrc.getTracks().array.forEach(element => {
                    element.stop();
                });
            }

            console.log("videoStart:" + this.video_start);
            if (this.video_start) {
                this.button_name = this.button_name_start;

                //カメラ出力中の場合
                // if (navigator && navigator.webKitGetUserMedia) {
                //     this.id_video.stop();

                //     //for (let i = 0; i < this.)

                //     this.videoStop();
                // }
                if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                    this.id_video.pause();
                    this.id_video.currentTime = 0;

                    this.videoStop(this.id_video);
                }

                if (this.videoCanvasCtx) {
                    this.videoCanvasCtx.clearRect(0, 0, this.video_width, this.video_height);
                }

                this.video_start = false;
                

            } else {
                this.id_video = this.$refs.ref_video;
                // this.canvasW = window.innerWidth 
                //     - (this.emotion.innerWidth + this.emotionBar.style.right) - 100;
                // this.canvasH = window.innerHeight 
                //     - this.message.innerHeight - 10;

                this.canvasW = this.video_width;
                this.canvasH = this.video_height;

                // if (navigator && navigator.webKitGetUserMedia) {
                //     console.log("videoStart: video");
                //     navigator.webKitGetUserMedia(
                //         {
                //             video:{
                //                 facingMode:"user", 
                //                 width:this.video_width,
                //                 height: this.video_height
                //             },
                //             audio: false
                //         },
                //         this._handleSuccess,
                //         this._handleerror
                //     );
                // } else {
                    console.log("videoStart: no navigator.webKitGetUserMedia");
                    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                        navigator.mediaDevices.getUserMedia({
                            video: {
                                facingMode:"user", 
                                width:this.video_width,
                                height: this.video_height
                            },
                            audio: false,
                        }).then(stream => {
                            this.handlesuccess(stream);
                        });
                    }
                //}
            }

        },
        videoStop: function(videoElem) {
            console.log("videoStop");
            const stream = videoElem.srcObject;
            const tracks = stream.getTracks();

            tracks.forEach(function(track) {
                track.stop();
            });

            videoElem.srcObject = null;

            if (this.videocanvas_interval) {
                for (let i = 0; i < this.videocanvas_interval; i++) {
                    clearInterval(this.videocanvas_interval[i]);
                }
            }
        },
        _handleSuccess: function() {
            console.log("_handleSuccess");
            if (this.videocanvas_interval) {
                for (let i = 0; i < this.videocanvas_interval; i++) {
                    clearInterval(this.videocanvas_interval[i]);
                }
            }

            console.log("_handleSuccess 2");
            this.videocanvas_interval = [];

            setTimeout(
                function(){
                    console.log("_handleSuccess 3");
                    this.id_video.play();
                }, 1000);

        },
        _handleerror: function() {
            console.log("_handleerror");
            this.alert("ビデオ失敗");
        },
        handlesuccess: function(stream) {
            console.log("handleSuccess");
            if (this.videocanvas_interval != null) {
                for (let i = 0; i < this.videocanvas_interval; i++) {
                    clearInterval(this.videocanvas_interval[i]);
                }
            }

            console.log("handleSuccess 2");
            this.videocanvas_interval = [];

            setTimeout(
                function(){
                    console.log("handleSuccess 3");
                    this.id_video.srcObject = stream;
                    this.id_video.play();
                }, 1000);

        },
        onVideoPlay : async function() {
            //ビデオ開始時
            console.log("onVideoPlay");

            this.button_name = this.button_name_end;

            this.video_start = true;

            if (this.videoCanvasCtx == null) {
                this.id_canvas = this.$refs.ref_canvas;

                this.videoCanvasCtx = this.id_canvas.getContext('2d');
            }

            console.log("this.videoCanvasCtx:" + this.videoCanvasCtx);

            const detectInterval = setInterval(async() => {
                const faceData = await faceapi.detectAllFaces(
                    this.id_video,
                    new faceapi.TinyFaceDetectorOptions()
                ).withFaceExpressions();

                console.log("this.videoCanvasCtx");
                console.log(this.videoCanvasCtx);

                if (this.videoCanvasCtx) {
                    console.log("this.videocanvasCtx clear");
                    this.videoCanvasCtx.clearRect(0, 0, this.video_width, this.video_height);
                    this.videoCanvasCtx.beginPath();
                }

                console.log("faceData");
                console.log(faceData);

                this.setSmile(faceData);
                
                const faceData_landmarks = await faceapi.detectAllFaces(
                    this.id_video, new faceapi.TinyFaceDetectorOptions()
                ).withFaceLandmarks();

                console.log("faceData_landmarks");
                console.log(faceData_landmarks);

                const iSize = {width: this.canvasW, height:this.canvasH};
                const rData = await faceapi.resizeResults(faceData_landmarks, iSize);

                this.drawReslut(rData);

            }, 300);

            this.videocanvas_interval.push(detectInterval);
        },
        setSmile: function(faceData) {

            if (faceData && faceData.length > 0 && faceData[0].expressions) {

                //笑顔の確認
                let happy = faceData[0].expressions.happy;
                let color = happy * 150 + 100;
            //console.log("happy:" + happy);
            //console.log("color:" + color);
                this.style_emotion = {
                                    bottom : (this.canvasH - 40) * happy + 'px',
                                    backgroundColor : `rgb(${color}, ${color}, 100)`
                                    };
                this.div_style_emotion = {
                    backgroundColor : `rgb(${color}, ${color}, 100)`
                };
                if(happy > 0.5){
                    this.txt_emotion = this.emotionTxt[0];
                } else {
                    this.txt_emotion = this.emotionTxt[1];
                }
            }
        },
        drawReslut: function(data) {
            for (let i = 0; i < data.length; i++) {
                const det = data[i].detection;
                const marks = data[i].landmarks.positions;
                const box = det.box;

                this.videoCanvasCtx.fillStyle = "red";
                this.videoCanvasCtx.strokeStyle = "red";

                this.videoCanvasCtx.lineWidth = 4;
                this.videoCanvasCtx.strokeRect(box.x, box.y, box.width, box.height);

                this.drawLandmarks(marks);
                this.drawNose(marks);
                this.drawGrasses(marks);
            }
        },
        drawLandmarks: function(marks) {

            //ランドマーク出力
            for(let i=0; i<marks.length; i++){
                let x = marks.x;
                let y = marks.y;
                this.videoCanvasCtx.fillRect(x, y, 2, 2);
                this.videoCanvasCtx.fillText(i, x, y, 18);
            }
        },
        drawNose: function(marks) {
            //鼻をつける

            this.videoCanvasCtx.strokeStyle = "black";
            this.videoCanvasCtx.lineWidth = 2;
            this.videoCanvasCtx.beginPath();

            for(let i=27; i<35; i++){
                let fX = marks[i].x;
                let fY = marks[i].y;
                let tX = marks[i+1].x;
                let tY = marks[i+1].y;
                this.videoCanvasCtx.moveTo(fX, fY);
                this.videoCanvasCtx.lineTo(tX, tY);
            }
            this.videoCanvasCtx.stroke();
        },
        drawGrasses: function(marks) {
            //グラスを書く

            this.videoCanvasCtx.strokeStyle = "black";
            this.videoCanvasCtx.lineWidth = 2;
            this.videoCanvasCtx.beginPath();
            this.videoCanvasCtx.moveTo(marks[39].x, marks[39].y);
            this.videoCanvasCtx.lineTo(marks[42].x, marks[42].y);
            this.videoCanvasCtx.stroke();

            // 左
            const lX = (marks[39].x + marks[36].x)/2;
            const lY = (marks[41].y + marks[38].y)/2;
            const lR = (marks[39].x - marks[36].x);
            this.videoCanvasCtx.beginPath();
            this.videoCanvasCtx.arc(lX, lY, lR, 0, Math.PI*2);
            this.videoCanvasCtx.stroke();
            // 右
            const rX = (marks[45].x + marks[42].x)/2;
            const rY = (marks[46].y + marks[43].y)/2;
            const rR = (marks[45].x - marks[42].x);
            this.videoCanvasCtx.beginPath();
            this.videoCanvasCtx.arc(rX, rY, rR, 0, Math.PI*2);
            this.videoCanvasCtx.stroke();
        }
    }
}

</script>
<style>
 .btn_div {
    height: inherit;
 }

 .span_emotion {
    font-size: 30px;
 }

 .emoticonBar {
    position: absolute;
    top: 0;
    right: 25;
    width: 40px;
    height: 480px;
    border-right: #DDD solid 6px;
 }

 .emotion {
    position: absolute;
    bottom: 0;
    right: 0;
    margin-top: 20px;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background-color: #CCC;
    text-align: center;
    font-size: 24px;
    line-height: 40px;
    transform: rotate(90deg);
    transition: bottom 0.3s;
 }

 .cameraArea {
    position: relative;
    margin: 0 auto;
 }

 .canvas {
    position: absolute;
    left: 0;
    top: 0;
 }

</style>