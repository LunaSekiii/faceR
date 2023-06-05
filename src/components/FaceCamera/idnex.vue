<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import {
	loadTinyFaceDetectorModel,
	loadFaceLandmarkTinyModel,
	loadFaceExpressionModel,
	loadAgeGenderModel,
	detectAllFaces,
	TinyFaceDetectorOptions,
	createCanvasFromMedia,
	resizeResults,
	draw,
} from "face-api.js";

// defineProps<{ msg: string }>();
const video = ref();
const box = ref();
const width = ref(0);
const height = ref(0);
const ModelPath = "./model/";

onMounted(() => {
	loadModel();
});

watchEffect(() => {
	width.value = video.value?.width || 480;
	height.value = video.value?.height || 640;
}, [video]);

const getCamera = async () => {
	try {
		const mediaStream = await navigator.mediaDevices.getUserMedia({
			video: true,
		});
		video.value.srcObject = mediaStream;
	} catch (e) {
		console.log("e", e);
	}
};

const loadModel = async () => {
	await loadTinyFaceDetectorModel(ModelPath);
	await loadFaceLandmarkTinyModel(ModelPath);
	await loadFaceExpressionModel(ModelPath);
	await loadAgeGenderModel(ModelPath);
	getCamera();
};

const detecetFace = () => {
	const canvas = createCanvasFromMedia(video.value);
	canvas.className = "canvas";
	const ctx = canvas.getContext("2d");
	box.value.append(canvas);
	// document.body.append(canvas);
	setInterval(async () => {
		const detections = await detectAllFaces(
			video.value,
			new TinyFaceDetectorOptions()
		)
			.withFaceLandmarks(true)
			.withFaceExpressions()
			.withAgeAndGender();
		const resizeDetections = resizeResults(detections, {
			width: width.value,
			height: height.value,
		});
		ctx?.clearRect(0, 0, width.value, height.value);
		draw.drawDetections(canvas, resizeDetections);
		draw.drawFaceLandmarks(canvas, resizeDetections);
		draw.drawFaceExpressions(canvas, resizeDetections);
		console.log("detections", detections);
	}, 100);
};
</script>

<template>
	<div ref="box" class="box">
		<video
			src=""
			id="video"
			ref="video"
			autoplay
			muted
			@play="detecetFace"
		></video>
	</div>
</template>

<style>
.box {
	position: relative;
}
.canvas {
	position: absolute;
	left: 0;
}
</style>
