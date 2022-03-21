<script>
	// initial side
	let cost = 100000
	let mortgage_period = 360 // period in months
	let dp_percentage = 0
	let fixed_rate = 0.06
	let floating_rate = 0
	let fixed_period = 360 // period dimana mortage fixed
	let floating_period = 0 // we don't count this

	// result side
	let dp_result = 0
	let remaining_mortgage = 0
	let remaining_mortgage_table = 0
	let total_fixed_payment = 0
	let total_floating_payment = 0
	let forTable = []
	let payment_each_month = 0
	let interest_installments = 0
	let principal_installments = 0
	let grand_sum = 0;
	let selected;

	function numberWithCommas(x) {
		return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
	}

	function getPaymentEachMonth(rem_mor, rate, mor_period) {
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

	$: {
		dp_result = cost * dp_percentage / 100
		remaining_mortgage = cost - dp_result
		// payment each month equation
		// period = 10 years => 120 months

		// buat menghitung
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
			let rate = index < fixed_period ? fixed_rate : floating_rate
			let period = index < fixed_period ? mortgage_period : floating_period
			
			let temp_payment_each_month = getPaymentEachMonth(remaining_mortgage, rate, period)

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
				total_fixed_payment += temp_interest_installments
				grand_sum += temp_interest_installments
				grand_sum = Number(grand_sum).toFixed(2)
			} else {
				total_floating_payment += temp_interest_installments
				grand_sum += temp_interest_installments
				grand_sum = Number(grand_sum).toFixed(2)
			}
		}

		total_fixed_payment = Number(total_fixed_payment).toFixed(2)
	}
</script>

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
			<input type="text" readonly>
			<label for="total_payment">Total Payment</label>
			<input id="total_payment" type="text" readonly>
			<label for="currency_symbol">Selected Currency</label>
			<select id="currency_symbol" bind:value={selected} on:change={() => console.log(selected)}>
				<option value="Rp. ">IDR</option>
				<option value="$">USD</option>
			</select>
			<button on:click={resetEverything}>Reset</button>
		</div>
	</div>

	<div class="table_side">
		<table class="result_table">
			<thead>
				<tr>
					<th>Year</th>
					<th>Payment Period (In Months)</th>
					<th>Payment Each Month</th>
					<th>Remaining Mortgage</th>
					<th>Principal Installment</th>
					<th>Interest Installment</th>
				</tr>
			</thead>
			<tbody>
				{#each forTable as item}
				<tr>
					<td>{item.year}</td>
					<td>{item.payment_period}</td>
					<td><b>{selected}</b>{(Number(item.payment_each_month).toFixed(2))}</td>
					<td><b>{selected}</b>{(Number(item.remaining_mortgage).toFixed(2))}</td>
					<td><b>{selected}</b>{(Number(item.interest_installments).toFixed(2))}</td>
					<td><b>{selected}</b>{(Number(item.principal_installments).toFixed(2))}</td>
				</tr>
				{/each}
			</tbody>
		</table>
	</div>

</div>

<style>
	.main_menu {
		display: grid;
		grid-template-areas:		
		"a a"
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
		grid-template-areas: 'a b';
		margin: 25px 100px;
		/* background-color:; */
	}

	.initial_side {
		grid-area: a;
		/* float: right; */
		/* border: 2px red solid; */
		margin-left: auto;
		margin-right: 50px;
	}

	.result_side {
		grid-area: b;
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

</style>