<script setup>
	import { onMounted, ref } from "vue";
	import SockJS from "sockjs-client/dist/sockjs";
	import { Client } from "@stomp/stompjs";

	// common
	// const baseUrl = "http://localhost:8080";
	const baseUrl = "https://test.datanucleusinc.com/greensky396";

	const totalTankVolume = ref(150);
	const grads = ref([]);

	// Balance Websocket
	const bankClient = ref(null);
	const companyId = ref("abcltd");
	const balance = ref("1009");
	const balanceIsConnected = ref(false);
	const balEndpoint = ref(`credit-balance?companyId=${companyId.value}`);
	const balanceDes = ref(
		`credit-balance/update-balance/companyId/${companyId.value}`
	);

	function toggleBalanceSocket() {
		const header = {
			companyId: companyId.value,
		};
		if (!balanceIsConnected.value) {
			webSocket(
				bankClient,
				balEndpoint.value,
				balanceDes.value,
				header,
				balanceIsConnected
			);
		} else {
			balanceIsConnected.value = false;
			bankClient.value.deactivate();
		}
	}

	// Tank Level
	const tankClient = ref(null);
	const tankDeviceId = ref({
		deviceId: "DEjR8Ktps7y9j9XEG",
		manufDeviceId: "389385858585582",
	});
	const tankIsConnected = ref(false);
	const tankLevel = ref("0");
	const tankEndpoint = ref(
		`tank-level?tankDeviceId=${tankDeviceId.value.deviceId}`
	);
	const tankDes = ref(
		`tank-level/new-level/device-id/${tankDeviceId.value.deviceId}`
	);

	function fillTank(value) {
		value = (Number(value) / totalTankVolume.value) * 300 + "px";
		document.querySelector(".water").style.cssText = `height: ${value};`;

		document.querySelector("#wave").classList.add("blinker", "text-danger");

		setTimeout(() => {
			document
				.querySelector("#wave")
				.classList.remove("blinker", "text-warning");
		}, 5000);
	}

	function toggleTankSocket() {
		console.log("Watching Tank: " + tankDeviceId.value.deviceId);
		const tankHeader = {
			tankDeviceId: tankDeviceId.value.deviceId,
		};
		if (!tankIsConnected.value) {
			webSocket(
				tankClient,
				tankEndpoint.value,
				tankDes.value,
				tankHeader,
				tankIsConnected
			);
		} else {
			tankIsConnected.value = false;
			tankClient.value.deactivate();
		}
	}

	//TRACKING DEVICE
	const commandResponse = ref("");
	const trackingDevice = ref([]);
	const trackerClient = ref(null);
	const tracker = ref({
		deviceId: "DEMWidw8NECrS6xHe",
		manufDeviceId: "867232054802104",
	});
	const customCommand = ref("*HQ,###,");
	// 867232054801403   867232054802104

	const speedMessage = ref({
		action: "speed-limit",
		manufDeviceId: tracker.value.manufDeviceId,
		deviceId: tracker.value.deviceId,
		speedLimit: 0,
	});

	const fenceData = ref({
		action: "set-geofence",
		deviceId: "DEMWidw8NECrS6xHe",
		manufDeviceId: "867232054802104",
		geoFenceName: "eeeeee",
		geoFenceStatus: "IN",
		shape: {
			points: [
				{
					index: 0,
					lat: 7.388309458015921,
					lng: 3.877561734432229,
				},
				{ index: 1, lat: 7.388309458015921, lng: 3.8785702450218285 },
				{ index: 2, lat: 7.386734770870718, lng: 3.8785702450218285 },
				{ index: 3, lat: 7.386734770870718, lng: 3.877561734432229 },
			],
			name: "RECTANGLE",
			code: "44",
			height: 175.29337116272808,
			width: 111.33478060893484,
		},
	});

	const trackerIsConnected = ref(false);
	const trackerEndpoint = ref(
		`tracker?trackingDeviceId=${tracker.value.deviceId}`
	);
	const trackerDes = ref(
		`tracker/live-data/device-id/${tracker.value.deviceId}`
	);

	function sendAdminCommand() {
		// 867232054801403   867232054802104
		const message = {
			action: "admin-numbers",
			manufDeviceId: tracker.value.manufDeviceId,
			deviceId: tracker.value.deviceId,
			numbers: ["08054815965", "09033915161", "08157868666"],
		};

		sendMessage(trackerClient, JSON.stringify(message));
	}

	function sendFenceCommand() {
		// 867232054801403   867232054802104
		const message = fenceData.value;

		sendMessage(trackerClient, JSON.stringify(message));
	}

	function sendHTBT() {
		// 867232054801403   867232054802104
		const message = {
			action: "heart-beat",
			manufDeviceId: tracker.value.manufDeviceId,
			deviceId: tracker.value.deviceId,
		};

		sendMessage(trackerClient, JSON.stringify(message));
	}

	function sendStatusCommand() {
		// 867232054801403
		const message = {
			action: "device-state",
			manufDeviceId: tracker.value.manufDeviceId,
			deviceId: tracker.value.deviceId,
			type: "SOFTWARE_VERSION",
		};

		sendMessage(trackerClient, JSON.stringify(message));
	}

	function sendSpeedLimitCommand() {
		// 867232054801403
		sendMessage(trackerClient, JSON.stringify(speedMessage.value));
	}

	function sendCustomCommand() {
		let cmd = customCommand.value
			.split("###")
			.join(tracker.value.manufDeviceId);

		cmd = `test-acyion##${tracker.value.manufDeviceId}##${cmd}`;
		sendMessage(trackerClient, cmd);
	}

	function setNewMessage(message) {
		commandResponse.value = "Incoming message...";
		setTimeout(() => {
			commandResponse.value = message;
		}, 2000);
	}

	function sendMessage(client, messageBody) {
		const destination = "/ws/tracker-command"; // Specify the destination where you want to send the message

		// Send the message
		client.value.publish({ destination, body: messageBody });
		console.log("Sent");
	}

	function toggleTrackerSocket() {
		console.log("Tracking Device: " + tracker.value.deviceId);
		const trackerHeader = {
			trackingDeviceId: tracker.value.deviceId,
			connectionType: "tracker",
		};
		if (!trackerIsConnected.value) {
			webSocket(
				trackerClient,
				trackerEndpoint.value,
				trackerDes.value,
				trackerHeader,
				trackerIsConnected
			);
		} else {
			trackerIsConnected.value = false;
			trackerClient.value.deactivate();
		}
	}

	function webSocket(client, endpoint, destination, header, indicator) {
		client.value = new Client({
			// brokerURL: ``,
			connectHeaders: header,
			debug: function (str) {
				// console.log("Debug: ", str);
			},
			webSocketFactory() {
				const socket = new SockJS(`${baseUrl}/${endpoint}`);

				return socket;
			},
			reconnectDelay: 5000,
			heartbeatIncoming: 4000,
			heartbeatOutgoing: 4000,
		});

		// console.log(typeof WebSocket)
		// Fallback code
		if (typeof WebSocket !== "function") {
			// For SockJS you need to set a factory that creates a new SockJS instance
			// to be used for each (re)connect
			// client.webSocketFactory = function () {
			//   // Note that the URL is different from the WebSocket URL
			//   return new WebSocket(SockJS('http://localhost:15674/stomp'));
			// };
			client.value.subscribe(
				`/ws/${destination}`,
				(content) => {
					console.log("content", content);
					indicator.value = true;
					if (header.companyId) {
						const credit = JSON.parse(content.body);
						console.log("fff Fallback", credit);
						balance.value = credit.creditBalance;
						console.log(balance.value);
					} else {
						tankLevel.value = content.body;
					}
				},
				header
			);
		}

		client.value.onConnect = (frame) => {
			indicator.value = true;
			if (header.companyId) {
				console.log("Connected to balance!");
			} else if (header.tankDeviceId) {
				console.log("Connected to Tank!");
			} else {
				console.log("Connected to Tracker!");
			}
			// "creditBalance":"4498.80"
			// console.log(frame)
			// Do something, all subscribes must be done is this callback
			// This is needed because this will be executed after a (re)connect
			client.value.subscribe(
				`/ws/${destination}`,
				(content) => {
					// console.log("content", content);

					if (header.companyId) {
						const credit = JSON.parse(content.body);
						balance.value = credit.creditBalance;
						console.log("Credit Balance: ", balance.value);
					} else if (header.tankDeviceId) {
						let value = content.body.split("=")[1];
						tankLevel.value = value;
						console.log("Tank Level: " + value);
						// let value = 250;

						// calculate hieght to fit cylinder div (300) in px
						fillTank(value);
					} else {
						const trackerData = JSON.parse(content.body);
						trackingDevice.value.push(trackerData);
						console.log("Tracker Data: " + trackerData);
					}
				},
				{ ...header }
			);

			client.value.subscribe(
				`/ws/tracker-command-response/${tracker.value.manufDeviceId}`,
				(content) => {
					console.log("content body: ", content.body);
					// indicator.value = true;
					setNewMessage(content.body);
				},
				header
			);
		};

		client.value.onStompError = function (frame) {
			// Will be invoked in case of error encountered at Broker
			// Bad login/passcode typically will cause an error
			// Complaint brokers will set `message` header with a brief message. Body may contain details.
			// Compliant brokers will terminate the connection after any error
			console.log("Broker reported error: " + frame.headers["message"]);
			console.log("Additional details: " + frame.body);
		};

		client.value.activate();
	}

	function graduate(tankVolume) {
		let cal = [];
		for (let i = 0; i <= tankVolume; i += tankVolume / 4) {
			cal.unshift(i);
		}

		totalTankVolume.value = tankVolume;

		grads.value = cal;
	}

	onMounted(() => {
		console.log("Websocket");
		let vol = 20;
		graduate(totalTankVolume.value);
		fillTank(15);
	});
</script>

<template>
	<div class="container p-md-5 mb-5 pt-3">
		<div class="">
			<div class="card mx-auto border-0">
				<div
					class="card-header bg-transparent shadow border-0 p-4 rounded-3"
				>
					<h1 class="mb-5 text-center">Websocket Test</h1>

					<div class="d-flex justify-content-end align-items-end">
						<div class="d-flex flex-column align-items-end">
							<div>
								<span
									:class="
										balanceIsConnected
											? 'text-success blinker'
											: 'text-muted'
									"
									style="font-size: small"
									class="me-3"
								>
									<i class="bi bi-circle-fill"></i>
								</span>
								<span class="text-end h3">
									<span>${{ balance }}</span>
								</span>
							</div>
							<span>Balance</span>
							<div class="mt-2 yme-auto">
								<button
									@click="toggleBalanceSocket()"
									:class="
										balanceIsConnected
											? 'btn-danger'
											: 'btn-success'
									"
									class="btn btn-sm"
								>
									<span v-if="!balanceIsConnected" class=""
										>Connect Balance</span
									>
									<span v-else class="">Disconnect</span>
								</button>
							</div>
						</div>
					</div>
				</div>
				<div class="card-body">
					<div class="row row-cols-2 g-3 align-items-stretched">
						<div class="col-12 col-md-6 h-100">
							<div class="card w-100 h-100">
								<div
									class="card-body d-flex flex-column align-items-end"
								>
									<div
										class="mb-5 w-100 d-flex justify-content-between w-100"
									>
										<span
											:class="
												tankIsConnected
													? 'text-success blinker'
													: 'text-muted'
											"
											style="font-size: small"
											class=""
										>
											<i class="bi bi-broadcast"></i>
										</span>

										<div class="d-flex align-items-end">
											<div class="cylinder">
												<div class="water"></div>
											</div>
											<div class="outer h-100">
												<div
													class="grad-scale"
													v-for="val in grads"
												>
													- {{ val }}
												</div>
											</div>
										</div>

										<div>
											<h5 class="text-end">
												<span>
													<i
														id="wave"
														class="bi bi-soundwave"
														style="
															font-size: smaller;
														"
													></i>
													{{ tankLevel }}</span
												>
											</h5>
											<span>Tank Level</span>
										</div>
									</div>
									<div class="w-100">
										<button
											@click="toggleTankSocket()"
											:class="
												tankIsConnected
													? 'btn-danger'
													: 'btn-success'
											"
											class="btn w-100"
										>
											<span
												v-if="!tankIsConnected"
												class=""
												>Connect Tank</span
											>
											<span v-else class=""
												>Disconnect</span
											>
										</button>
									</div>
								</div>
							</div>
						</div>
						<div class="col-12 col-md-6">
							<div class="card w-100 h-100">
								<div
									class="card-body d-flex flex-column align-items-end"
								>
									<div
										class="mb-5 row justify-content-between w-100"
									>
										<div class="col">
											<span
												:class="
													trackerIsConnected
														? 'text-success blinker'
														: 'text-muted'
												"
												style="font-size: small"
												class=""
											>
												<i
													class="bi bi-geo-alt-fill"
												></i>
											</span>
										</div>
										<div class="col">
											<div
												class="text-end text-truncatei"
											>
												<span
													>({{
														tracker.deviceId
													}})</span
												>
											</div>
											<span>Tracker</span>
										</div>
									</div>
									<div class="w-100 table-responsive">
										<table
											class="table table-striped table-hover"
										>
											<thead>
												<tr>
													<th>Latitude</th>
													<th>Longitude</th>
													<th>Time</th>
													<th>Status</th>
												</tr>
											</thead>
											<tbody>
												<tr
													v-for="data in trackingDevice"
												>
													<td>{{ data.lat }}</td>
													<td>{{ data.lng }}</td>
													<td>{{ data.time }}</td>
													<td>
														Stopped:
														{{
															data.status.stopped
														}}
													</td>
												</tr>
											</tbody>
										</table>
									</div>
									<div class="w-100">
										<button
											@click="toggleTrackerSocket()"
											:class="
												trackerIsConnected
													? 'btn-danger'
													: 'btn-success'
											"
											class="btn w-100"
										>
											<span
												v-if="!trackerIsConnected"
												class=""
												>Start Trackering</span
											>
											<span v-else class=""
												>Disconnect</span
											>
										</button>
									</div>
								</div>
							</div>
						</div>
						<div class="col-12">
							<div class="card w-100 h-100">
								<div
									class="card-body d-flex flex-column align-items-eend"
								>
									<div>
										{{ commandResponse }}
									</div>
									<div class="d-flex mt-3 align-items-end">
										<div>
											<button
												@click="sendStatusCommand()"
												class="btn btn btn-sm btn-outline-primary"
											>
												Status command
											</button>
										</div>
										<div class="mx-2">
											<button
												@click="sendAdminCommand()"
												class="btn btn-sm btn-outline-primary"
											>
												Admin command
											</button>
										</div>

										<div>
											<input
												class="form-control"
												type="text"
												v-model="
													speedMessage.speedLimit
												"
												placeholder="Enter speed limit"
											/>
											<div class="mt-2">
												<button
													@click="
														sendSpeedLimitCommand()
													"
													class="btn btn-outline-primary"
												>
													Set limit
												</button>
											</div>
										</div>
									</div>
									<div class="d-flex mt-3 align-items-end">
										<div>
											<button
												@click="sendHTBT()"
												class="btn btn btn-sm btn-outline-primary"
											>
												Send HTBT
											</button>
										</div>
										<div class="mx-2">
											<button
												@click="sendFenceCommand()"
												class="btn btn-sm btn-outline-primary"
											>
												Set Fence
											</button>
										</div>
									</div>
									<!-- Custom command test -->
									<div>
										<h6 class="text-center mt-5">
											Custom command
										</h6>
										<input
											class="form-control"
											type="text"
											v-model="customCommand"
											placeholder="Enter command"
										/>
										<div class="mt-3">
											<button
												@click="sendCustomCommand()"
												class="btn btn-outline-primary w-100"
											>
												Send Command
											</button>
										</div>
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
	.cylinder {
		outline: 2px solid #f00;
		position: relative;
		overflow: hidden;
		margin: 0 auto;
		width: 100px;
		height: 300px;
		border: 2px solid gray;
		border-radius: 50px/25px;
		background-color: rgba(160, 160, 160, 0.5);
	}

	.cylinder:before {
		position: absolute;
		margin-left: -1px;
		margin-top: -5px;
		left: 0;
		top: 0;
		width: 100px;
		height: 50px;
		border-radius: 50px/25px;
		background-color: grey;
		content: "";
	}

	.cylinder:after {
		margin-left: -1px;
		margin-bottom: -5px;
		position: absolute;
		left: 0;
		bottom: 0;
		width: 100px;
		height: 50px;
		border-radius: 50px/25px;
		background-color: rgba(0, 160, 160, 0.9);
		content: "";
	}

	.water {
		margin-bottom: -5px;
		position: absolute;
		left: 0;
		bottom: 0;
		width: 100px;
		height: 0px;
		padding-top: 50px;
		border-radius: 50px/25px;
		background-color: rgba(0, 160, 160, 0.9);
		transition: 0.3s linear;
	}

	.water:before {
		position: absolute;
		left: 0;
		top: 0;
		width: 100px;
		height: 50px;
		border-radius: 50px/25px;
		background-color: rgba(0, 160, 160, 0.2);
		content: "";
	}

	.water:after {
		position: absolute;
		left: 0;
		bottom: 0;
		width: 100px;
		height: 50px;
		border-radius: 50px/25px;
		background-color: rgba(0, 160, 160, 0.4);
		content: "";
	}
	.blinker {
		animation: blink 1s linear infinite;
	}
	@keyframes blink {
		0% {
			opacity: 0.2;
		}
		25% {
			opacity: 0.3;
		}
		50% {
			opacity: 0.5;
		}
		75% {
			opacity: 0.8;
		}
		100% {
			opacity: 1;
		}
	}

	.grad-scale {
		margin-left: 10px;
		height: 0px;
		margin-top: 75px;
	}
</style>
