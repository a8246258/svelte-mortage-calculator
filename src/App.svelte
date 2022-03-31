<script lang="ts">
	// interfaces
	interface TableRow{
		year: number,
		payment_period: number,
		remaining_mortgage: number,
		payment_each_month: number,
		principal_installments: number,
		interest_installments: number,
	}
	// forTable[0] = {
	// 		"year": 1,
	// 		"payment_period": 1,
	// 		"remaining_mortgage": remaining_mortgage_table,
			// payment_each_month,
			// principal_installments,
			// interest_installments,
	// 	}

	// initial side
	let cost: number = 300000
	let mortgage_period: number = 240 // period in months
	let dp_percentage: number = 10
	let fixed_rate: number = 0.05
	let floating_rate: number = 0.11
	let fixed_period: number = 60 // period dimana mortage fixed
	let floating_period: number = mortgage_period - fixed_period // we don't count this

	// result side
	let dp_result : number = 0
	let remaining_mortgage : number = 0
	let remaining_mortgage_table : number = 0
	let total_fixed_payment : number = 0
	let total_floating_payment : number = 0
	let forTable = []
	let yearlyTable = []
	let payment_each_month : number = 0
	let interest_installments : number = 0
	let principal_installments : number = 0
	let grand_sum : number = 0;
	let selected : string;

	function numberWithCommas(x: string): string {
		return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
	}

	function getPaymentEachMonth(rem_mor: number, rate: number, mor_period: number): number {
		// console.log("rem_mor: "+rem_mor)
		return (rem_mor * (rate / 12)) / (1 - 1 / Math.pow(1 + rate / 12, mor_period))
	}

	function resetEverything() {
		// initial side
	 	cost = 0
	 	mortgage_period = 0 // period in months
	 	dp_percentage = 0
	 	fixed_rate = 0
	 	floating_rate = 0
	 	fixed_period = 0 // period dimana mortage fixed
	 	floating_period = 0 // we don't count this

	// result side
	 	dp_result = 0
	 	remaining_mortgage = 0
	 	remaining_mortgage_table = 0
	 	total_fixed_payment = 0
	 	total_floating_payment = 0
	 	forTable = []
	 	payment_each_month = 0
	 	interest_installments = 0
	 	principal_installments = 0
	 	grand_sum = 0;
	}

	function changeTable(index: number) {
		yearlyTable = []
		// console.table(forTable)
		// console.log("sent index: "+index)
		let multiplier = 12*(index-1) // 12
		let upperLimit = (12*index)-1 // 23
		
		// console.log(multiplier)
		// console.log(upperLimit)
		
		for (multiplier; multiplier <= upperLimit; multiplier++) {
			// yearlyTable.push(forTable[multiplier])
			// yearlyTable = yearlyTable
			yearlyTable[11 - (upperLimit - multiplier)] = forTable[multiplier]
		}
		
		// console.log(yearlyTable)
	}

	$: {
		floating_period = mortgage_period - fixed_period
		dp_result = cost * dp_percentage / 100
		remaining_mortgage = cost - dp_result
		// payment each month equation
		// period = 10 years => 120 months

		// buat menghitung

		if (cost != 0 && mortgage_period  != 0 && fixed_rate  != 0 && floating_rate  != 0 && fixed_period  != 0) {

			payment_each_month = getPaymentEachMonth(remaining_mortgage, fixed_rate, mortgage_period)
	
			interest_installments = (remaining_mortgage * fixed_rate / 12)
			
			principal_installments = (payment_each_month - interest_installments)
	
			remaining_mortgage_table = remaining_mortgage - principal_installments
	
			forTable = []
			forTable[0] = {
				"year": 1,
				"payment_period": 1,
				"remaining_mortgage": remaining_mortgage_table,
				payment_each_month,
				principal_installments,
				interest_installments,
			}
	
			total_fixed_payment = 0
			total_fixed_payment += interest_installments
	
			// makeDataForTable()
			for (let index = 1; index < mortgage_period; index++) {
				let remaining_payment = index < fixed_period ? remaining_mortgage : forTable[fixed_period-1].remaining_mortgage
				let rate = index < fixed_period ? fixed_rate : floating_rate
				let period = index < fixed_period ? mortgage_period : floating_period
				// console.log("index: " + index)
				// console.log("rate: " + rate)
				// console.log("period: " +period)
				
				let temp_payment_each_month = getPaymentEachMonth(remaining_payment, rate, period)
	
				let temp_interest_installments = (forTable[index-1].remaining_mortgage * rate / 12)
				
				let temp_principal_installments = (temp_payment_each_month - temp_interest_installments)
	
				let temp_remaining_mortgage_table = Math.abs(forTable[index-1].remaining_mortgage - temp_principal_installments)
	
				forTable[index] = {
					"year": Math.floor(index/12) + 1,
					"payment_period": index + 1,
					"remaining_mortgage": temp_remaining_mortgage_table,
					"payment_each_month": temp_payment_each_month,
					"principal_installments": temp_principal_installments,
					"interest_installments": temp_interest_installments,
				}
	
				if (index < fixed_period) {
					total_fixed_payment += temp_payment_each_month
					// grand_sum += temp_interest_installments
					// grand_sum = Number(grand_sum).toFixed(2)
				} else {
					total_floating_payment += temp_payment_each_month
					// grand_sum += temp_interest_installments
					// grand_sum = Number(grand_sum).toFixed(2)
				}
			}
	
			total_fixed_payment = +Number(total_fixed_payment).toFixed(2)
			total_floating_payment = +Number(total_floating_payment).toFixed(2)
			grand_sum = +Number(total_fixed_payment+total_floating_payment).toFixed(2)
		}
		
	}
</script>

<svelte:head>
	<title>Kalkulator KPR</title>
	<html lang="en"/>
</svelte:head>

<div class='main_menu'>

	<div class="top_side">
		<div class='initial_side'>
			<label for="">Property Cost</label>
			<input type="number" bind:value={cost}>
			<label for="">Mortgage Period</label>
			<input type="text" bind:value={mortgage_period}>
			<label for="">Down Payment Percentage</label>
			<input type="text" bind:value={dp_percentage}>
			<label for="">Fixed Rate</label>
			<input type="text" bind:value={fixed_rate}>
			<label for="">Floating Rate</label>
			<input type="text" bind:value={floating_rate}>
			<label for="">Fixed Period</label>
			<input type="text" bind:value={fixed_period}>
			<label for="">Floating Period</label>
			<input type="text" readonly bind:value={floating_period}>
		</div>
		
		<div class="result_side">
			<label for="">Down Payment</label>
			<input id="" type="text" bind:value={dp_result} readonly>
			<label for="">Remaining Mortgage</label>
			<input id="" type="text" bind:value={remaining_mortgage} readonly>
			<label for="">Total Fixed Interest</label>
			<input id="" type="text" bind:value={total_fixed_payment} readonly>
			<label for="">Total Floating Payment</label>
			<input type="text" bind:value={total_floating_payment} readonly>
			<label for="total_payment">Total Payment</label>
			<input id="total_payment" type="text" bind:value={grand_sum} readonly>
			<label for="currency_symbol">Selected Currency</label>
			<select id="currency_symbol" bind:value={selected}>
				<option value="Rp. ">IDR</option>
				<option value="$">USD</option>
			</select>
			<button on:click={resetEverything}>Reset</button>
		</div>
	</div>

	
	<div class="table_side">
		<div class="table_buttons">
			{#each Array(Math.floor(mortgage_period / 12)) as item, i}
			<button on:click={() => {changeTable(i+1)}}>{i+1}</button>
			{/each}
		</div>
		<table class="result_table">
			<thead>
				<tr>
					<th>Year</th>
					<th>Payment Period (In Months)</th>
					<th>Payment Each Month</th>
					<th>Remaining Mortgage</th>
					<th>Interest Installment</th>
					<th>Principal Installment</th>
				</tr>
			</thead>
			<tbody>
				{#each yearlyTable as item, i}
				<tr>
					{#if i % 12 === 0}
					<td rowspan="12">{item.year}</td>
					{/if}
					{#if item.payment_period % 10 === 1 && (item.payment_period < 10 || item.payment_period > 14)}
					<td><b>{item.payment_period}<sup>st</sup></b></td>
					{:else if item.payment_period % 10 === 2 && (item.payment_period < 10 || item.payment_period > 14)}
					<td><b>{item.payment_period}<sup>nd</sup></b></td>
					{:else if item.payment_period % 10 === 3 && (item.payment_period < 10 || item.payment_period > 14)}
					<td><b>{item.payment_period}<sup>rd</sup></b></td>
					{:else}
					<td><b>{item.payment_period}<sup>th</sup></b></td>
					{/if}
					<td><b>{selected}</b>{(Number(item.payment_each_month).toFixed(2))}</td>
					<td><b>{selected}</b>{(Number(item.remaining_mortgage).toFixed(2))}</td>
					<td><b>{selected}</b>{(Number(item.interest_installments).toFixed(2))}</td>
					<td><b>{selected}</b>{(Number(item.principal_installments).toFixed(2))}</td>
				</tr>
				{/each}
			</tbody>
		</table>

		<div class="table_buttons">
			{#each Array(Math.floor(mortgage_period / 12)) as item, i}
			<button on:click={() => {changeTable(i+1)}}>{i+1}</button>
			{/each}
		</div>
	</div>

</div>

<style>
	.main_menu {
		display: grid;
		grid-template-areas:		
		"a a"
		"b b"
		"c c";
		background-color: rgb(206, 206, 206);
	}
	
	label {
		/* color: #ffffff; */
		font-weight: 700;
		margin: 10px;
	}

	input {
		border-radius: 20px;
		padding: 10px 15px;
	}

	.top_side {
		grid-area: a;
		display: grid;
		grid-template-areas: '1 2';
		margin: 25px 100px;
		/* background-color:; */
	}

	.initial_side {
		grid-area: "1";
		/* float: right; */
		/* border: 2px red solid; */
		margin-left: auto;
		margin-right: 50px;
	}

	.result_side {
		grid-area: "2";
		/* border: 2px red solid; */
	}
	
	.table_side {
		grid-area: c;
		margin: 0px 35px;
	}

	.result_table {
		/* grid-area: c; */
		border: 1px solid black;
		width: 100%;
		border-collapse: collapse;
		margin: 25px 0;
		box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
	}

	.result_table th, td {
		border: 1px solid black;
		padding: 12px 15px;
	}

	.result_table thead th {
		background-color: #009879;
		color: #ffffff;
		text-align: left;
	}

	.result_table tbody tr:nth-child(odd) {
		background-color: #ffffff;
	}

	.result_table tbody tr:nth-child(even) {
		background-color: #f3f3f3;
	}

	.table_buttons {
		display: flex;
		flex-direction: row;
	}

	.table_buttons > button {
		margin-right: 5px;
		border-radius: 50%;
	}

</style>