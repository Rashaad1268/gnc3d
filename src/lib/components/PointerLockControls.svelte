<script lang="ts">
	import { createEventDispatcher, onDestroy } from "svelte";
	import { Euler, Camera, Group } from "three";
	import { useThrelte, useParent, useFrame } from "@threlte/core";
	// Set to constrain the pitch of the camera
	// Range is 0 to Math.PI radians
	export let minPolarAngle = 0; // radians
	export let maxPolarAngle = Math.PI; // radians
	export let pointerSpeed = 1.0;
	export let object: Group;
	export let plock: boolean;
	export let zooming: number;
	$: {
		if (zooming !== -1) {
			zooming > 1 && (plock = false);
		}
	}
	let isLocked = false;
	const { renderer, invalidate } = useThrelte();
	const domElement = renderer.domElement;
	const camera = useParent();
	const dispatch = createEventDispatcher();
	const _euler = new Euler(0, 0, 0, "YXZ");
	const _PI_2 = Math.PI / 2;
	if (!renderer) {
		throw new Error("Threlte Context missing: Is <PointerLockControls> a child of <Canvas>?");
	}
	const isCamera = (p: any): p is Camera => {
		return p.isCamera;
	};
	if (!isCamera($camera)) {
		throw new Error("Parent missing: <PointerLockControls> need to be a child of a <Camera>");
	}
	const onChange = () => {
		invalidate("PointerLockControls: change event");
		dispatch("change");
	};
	export const lock = () => domElement.requestPointerLock();
	export const unlock = () => document.exitPointerLock();
	domElement.addEventListener("mousemove", onMouseMove);
	domElement.addEventListener("touchmove", onTouchMove);
	domElement.addEventListener("wheel", onWheel);
	domElement.addEventListener("click", lock);
	domElement.addEventListener("touchend", onTouchEnd);
	domElement.ownerDocument.addEventListener("pointerlockchange", onPointerlockChange);
	domElement.ownerDocument.addEventListener("pointerlockerror", onPointerlockError);
	onDestroy(() => {
		domElement.removeEventListener("mousemove", onMouseMove);
		domElement.removeEventListener("touchmove", onTouchMove);
		domElement.removeEventListener("wheel", onWheel);
		domElement.removeEventListener("click", lock);
		domElement.removeEventListener("touchend", onTouchEnd);
		domElement.ownerDocument.removeEventListener("pointerlockchange", onPointerlockChange);
		domElement.ownerDocument.removeEventListener("pointerlockerror", onPointerlockError);
	});
	function onWheel(event: WheelEvent) {
		// console.log(event.deltaY / 16);
		// console.log(idealOffset.z)
		if (Math.floor(event.deltaY / 32) > 0) {
			dispatch("unlock");
			unlock();
			isLocked = false;
			plock = false;
		}
	}
	function onMouseMove(event: MouseEvent) {
		if (!isLocked) return;
		if (!$camera) return;
		const { movementX, movementY } = event;
		_euler.setFromQuaternion($camera.quaternion);
		_euler.y -= movementX * 0.002 * pointerSpeed;
		_euler.x -= movementY * 0.002 * pointerSpeed;
		_euler.x = Math.max(_PI_2 - maxPolarAngle, Math.min(_PI_2 - minPolarAngle, _euler.x));
		$camera.quaternion.setFromEuler(_euler);
		onChange();
	}
	function onTouchEnd(event: TouchEvent) {
		prev = null;
	}
	let prev: [number, number] | null = null;
	function onTouchMove(event: TouchEvent) {
		// console.log(event.touches)
		// event.preventDefault();
		if (!$camera) return;
		const touch = event.touches[event.touches.length - 1];
		let movementX = 0,
			movementY = 0;
		if (prev) {
			// be aware that these only store the movement of the first touch in the touches array
			movementX = touch.pageX - prev[0];
			movementY = touch.pageY - prev[1];
		}

		prev = [touch.pageX, touch.pageY];
		// console.log(movementX, movementY)
		_euler.setFromQuaternion($camera.quaternion);
		_euler.y -= movementX * 0.02 * pointerSpeed;
		_euler.x -= movementY * 0.02 * pointerSpeed;
		_euler.x = Math.max(_PI_2 - maxPolarAngle, Math.min(_PI_2 - minPolarAngle, _euler.x));
		$camera.quaternion.setFromEuler(_euler);
		onChange();
	}
	useFrame(() => {
		if (!$camera) return;
		$camera.position.copy(object.position);
	});
	function onPointerlockChange() {
		if (document.pointerLockElement === domElement) {
			dispatch("lock");
			isLocked = true;
		} else {
			dispatch("unlock");
			isLocked = false;
		}
	}
	function onPointerlockError() {
		console.error("PointerLockControls: Unable to use Pointer Lock API");
	}
</script>
