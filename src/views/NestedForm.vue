<script setup>
	import { ref } from "vue";
	import axios from "axios";

	const loading = ref(false);

	const tankModel = ref({
		consumption: 43,
		contentId: "AIR",
		delVolume: 23,
		filling: "0",
		lastReading: "0",
		minUsefVolume: 345,
		totalUsefVolume: 34,
		totalVolume: 10.450278625902733,
		height: 243,
		diameter: 234,
		color: "#000000",
		shape: "Cylinder",
	});

	const formData = ref({
		assetId: "new-record",
		assetName: "Fourth Tank Calibration Test Aj",
		assetDesc: "sff",
		assetTypeId: "TANK",
		companyId: "abcltd",
		locationId: "brampton",
		userCompanyId: "abcltd",
		userId: "CUKFW6DTty6G5P2Jt",
		statusId: "A",
		// tankCalibration: "(binary)",
	});

	const fileInput = ref();

	const handleSubmit = async () => {
		if (!fileInput.value.files.length) {
			// Handle missing file error
			console.error("Please select a tank calibration file.");
			return;
		}

		const file = fileInput.value.files[0];
		loading.value = true;

		const assetData = new FormData();

		// Add regular form fields:
		Object.entries(formData.value).forEach(([key, value]) =>
			assetData.append(key, value)
		);

		const tankData = JSON.stringify(tankModel.value);

		// Ensure you're sending the object directly, not JSON-stringified:
		assetData.append("assetTankModel", tankData);
		assetData.append("addDeviceList", "[]");
		assetData.append("delDeviceList", "[]");

		// Add file:
		assetData.append("tankCalibration", file);

		axios
			.post("http://localhost:8080/asset/save-asset", assetData, {
				headers: {
					"Content-Type": "multipart/form-data",
				},
			})
			.then((response) => {
				console.log("Response:", response.data);
			})
			.catch((error) => {
				console.error("Error:", error);
			})
			.finally(() => {
				loading.value = false;
			});
	};
</script>
<template>
	<div
		class="container vh-100 d-flex align-items-center justify-content-center"
	>
		<div class="card shadow h-50">
			<div class="card-body">
				<form action="" @submit.prevent="handleSubmit()">
					<h1 class="text-center text-muted mb-5">
						Save Asset Trial
					</h1>
					<div class="form-group mb-3">
						<label for="tankCalibration" class="form-label"
							>Tank Calibration File:</label
						>
						<input
							class="form-control"
							type="file"
							id="tankCalibration"
							accept=".xlsx, .xls, .csv"
							ref="fileInput"
							required
						/>
					</div>
					<button class="btn btn-primary w-100">
						<span v-if="loading" class="spinner-border"></span>
						<span v-else>Saved Asset</span>
					</button>
				</form>
			</div>
		</div>
	</div>
</template>
