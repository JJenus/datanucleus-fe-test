<script setup>
	import interact from "interactjs";
	import { onMounted, ref } from "vue";
	// localStorage.clear();
	const devices = ref([
		{
			id: "1",
			name: "Water",
			width: 100,
			height: 40,
			position: {
				x: 1,
				y: 0,
			},
		},
		{
			id: "2",
			name: "Door",
			width: 100,
			height: 40,
			position: {
				x: 0,
				y: 0,
			},
		},
		{
			id: "3",
			name: "Fire",
			width: 100,
			height: 40,
			position: {
				x: 0,
				y: 0,
			},
		},
	]);

	const setPosition = (id, position) => {
		for (let device of devices.value) {
			if (device.id === id) {
				device.position = position;
				break;
			}
		}

		localStorage.setItem("devices", JSON.stringify(devices.value));
	};

	// target elements with the "draggable" class
	interact(".draggable").draggable({
		// enable inertial throwing
		inertia: true,
		// keep the element within the area of it's parent
		modifiers: [
			interact.modifiers.restrictRect({
				restriction: "parent",
				endOnly: true,
			}),
		],
		// enable autoScroll
		autoScroll: true,

		listeners: {
			// call this function on every dragmove event
			move: dragMoveListener,

			// call this function on every dragend event
			end(event) {
				var target = event.target;
				// keep the dragged position in the data-x/data-y attributes
				var x =
					(parseFloat(target.getAttribute("data-x")) || 0) + event.dx;
				var y =
					(parseFloat(target.getAttribute("data-y")) || 0) + event.dy;

				const id = target.getAttribute("data-parrot-device-id");
				console.log("data-parrot-device-id: ", id, { x, y });

				setPosition(id, { x, y });
			},
		},
	});

	function dragMoveListener(event) {
		var target = event.target;
		// keep the dragged position in the data-x/data-y attributes
		var x = (parseFloat(target.getAttribute("data-x")) || 0) + event.dx;
		var y = (parseFloat(target.getAttribute("data-y")) || 0) + event.dy;

		// translate the element
		target.style.transform = "translate(" + x + "px, " + y + "px)";

		// update the posiion attributes
		target.setAttribute("data-x", x);
		target.setAttribute("data-y", y);
	}

	// this function is used later in the resizing and gesture demos
	window.dragMoveListener = dragMoveListener;

	onMounted(() => {
		const loadDevices = localStorage.getItem("devices");
		if (loadDevices) {
			console.log(JSON.parse(loadDevices));
			devices.value = JSON.parse(loadDevices);
		}
	});
</script>

<template>
	<div class="container">
		<h1>This is an about page</h1>

		<div class="border border-2 border-primary p-4">
			<div
				class="draggable card"
				v-for="device in devices"
				:style="`max-width: ${device.width}px; min-height: ${device.height}px; transform: translate(${device.position.x}px, ${device.position.y}px);`"
				:data-x="device.position.x"
				:data-y="device.position.y"
				:data-parrot-device-id="device.id"
				:id="device.id"
			>
				<div class="card-body text-center">
					{{ device.name }}
				</div>
			</div>
		</div>
	</div>
</template>

<style>
	.draggable {
		border-radius: 0.75em;
		touch-action: none;
		user-select: none;
		transform: translate(0px, 0px);
	}
</style>
