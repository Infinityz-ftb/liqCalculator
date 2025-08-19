<script>
	import colorLogo from '../assets/infinityLogo.png';

	let availableCapital = $state(0);
	let personalOrSMSF = $state('personal');
	let smsfCashNeeded = $state(0);
	let remainingDeposits = $state(0);
	let solicitorsFees = $state(0);
	let stampDuty = $state(0);
	let constructionCosts = $state(0);
	let contingencyCapital = $state(0);
	let settlementDate = $state('');
	let currentDateStr = new Date().toISOString().slice(0, 10);

	let monthsBetween = $derived.by(() => {
		if (!settlementDate) return 0;
		const current = new Date(currentDateStr);
		const settlement = new Date(settlementDate);

		const yearsDiff = settlement.getFullYear() - current.getFullYear();
		const monthsDiff = settlement.getMonth() - current.getMonth();

		let m = yearsDiff * 12 + monthsDiff;
		if (settlement.getDate() < current.getDate()) {
			m -= 1;
		}
		return m < 0 ? 0 : m;
	});

	let totalCapitalRequired = $derived(
		smsfCashNeeded +
			remainingDeposits +
			solicitorsFees +
			stampDuty +
			constructionCosts +
			contingencyCapital
	);

	let capitalRequired12Months = $derived(totalCapitalRequired);
	let capitalRequired24Months = $derived(totalCapitalRequired);

	let capitalGap = $derived.by(() => {
		if (monthsBetween >= 24) {
			// Don’t subtract – just show you have full available capital
			return -availableCapital;
		} else {
			// Normal gap calculation
			return totalCapitalRequired - availableCapital;
		}
	});

	function exportToText() {
		const data = `
Available Capital: ${availableCapital}
Personal or SMSF: ${personalOrSMSF}
SMSF Cash Needed: ${smsfCashNeeded}
Remaining Deposits: ${remainingDeposits}
Solicitors Fees: ${solicitorsFees}
Stamp Duty: ${stampDuty}
Construction Costs: ${constructionCosts}
Contingency Capital: ${contingencyCapital}
Settlement Date: ${settlementDate}
Months Between: ${monthsBetween}
Total Capital Required: ${totalCapitalRequired}
Capital Surplus: ${-capitalGap}
`;

		const blob = new Blob([data], { type: 'text/plain' });
		const link = document.createElement('a');
		link.href = URL.createObjectURL(blob);
		link.download = 'liquidity_calculator.txt';
		link.click();
		URL.revokeObjectURL(link.href);
	}
</script>

<main class="container mx-auto max-w-md p-8">
	<div class="flex w-full items-center justify-between py-2">
		<div id="pdf-content">
			<img src={colorLogo} alt="infinity-logo" class=" m-auto h-12" />
			<h1 class="mb-4 text-center text-2xl font-bold">The Liquidity Calculator</h1>

			<div class="space-y-4">
				<div>
					<label class="mb-2 block text-sm font-medium" for="available-capital"
						>How much capital do you have available?</label
					>
					<input
						type="number"
						bind:value={availableCapital}
						placeholder="e.g. 250000"
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<label class="mb-2 block text-sm font-medium" for="personal-or-smsf"
						>Personal or SMSF?</label
					>
					<select
						bind:value={personalOrSMSF}
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					>
						<option value="personal">Personal</option>
						<option value="smsf">SMSF</option>
					</select>
				</div>

				{#if personalOrSMSF === 'smsf'}
					<div>
						<label class="mb-2 block text-sm font-medium" for="smsf-cash-needed">
							How much cash are you required to maintain for liquidity?
						</label>
						<input
							type="number"
							bind:value={smsfCashNeeded}
							placeholder="e.g. 100000"
							class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
						/>
					</div>
				{/if}

				<div>
					<label class="mb-2 block text-sm font-medium" for="remaining-deposits"
						>Remaining deposits:</label
					>
					<input
						type="number"
						bind:value={remainingDeposits}
						placeholder="e.g. 100000"
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<label class="mb-2 block text-sm font-medium" for="solicitors-fees"
						>Solicitors fees:</label
					>
					<input
						type="number"
						bind:value={solicitorsFees}
						placeholder="e.g. 100000"
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<label class="mb-2 block text-sm font-medium" for="stamp-duty"
						>Stamp duty & other transfer costs:</label
					>
					<input
						type="number"
						bind:value={stampDuty}
						placeholder="e.g. 100000"
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<label class="mb-2 block text-sm font-medium" for="construction-costs"
						>Construction costs (if not applicable leave at 0):</label
					>
					<input
						type="number"
						bind:value={constructionCosts}
						placeholder="e.g. 100000"
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<label class="mb-2 block text-sm font-medium" for="contingency-capital"
						>Contingency capital:</label
					>
					<input
						type="number"
						bind:value={contingencyCapital}
						placeholder="e.g. 100000"
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<label class="mb-2 block text-sm font-medium" for="settlement-date"
						>Next projected property settlement date:</label
					>
					<input
						type="date"
						bind:value={settlementDate}
						class="w-full rounded-md border border-gray-300 p-3 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div class="space-y-2 rounded-md bg-gray-100 p-4">
					<p class="font-medium">
						Amount required for your current strategy: ${totalCapitalRequired}
					</p>
					<p>
						{#if monthsBetween <= 12 && monthsBetween >= 0}
							Capital required in 12 months: ${capitalRequired12Months.toLocaleString()}
						{/if}
					</p>
					<p>
						{#if monthsBetween >= 12}
							Capital required in 24 Months or Moree: ${capitalRequired24Months.toLocaleString()}
						{/if}
					</p>
					<p class={capitalGap > 0 ? 'font-semibold text-red-600' : 'font-semibold text-green-600'}>
						{capitalGap > 0
							? `You are short by $${capitalGap.toLocaleString()}`
							: `You have $${Math.abs(capitalGap).toLocaleString()} surplus available right now!`}
					</p>
				</div>

				<button
					onclick={() => exportToText()}
					class="mt-4 cursor-pointer rounded-md bg-blue-500 px-4 py-2 text-white"
				>
					Download Text File
				</button>
				<div>
					<p class=" text-sm text-red-400">
						Use of this calculator is indicative only, and does not constitute financial advice.
						Quality of results are subject to the accuracy of the information provided by the user.
					</p>
				</div>
			</div>
		</div>
	</div>
</main>

<style>
	:global(body) {
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
		margin: 0;
		background-color: #f9fafb;
	}
</style>
