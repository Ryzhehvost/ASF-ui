<template>
	<div class="bgr__summary">
		<h3 class="subtitle bgr__summary-title">{{ title }}</h3>

		<div class="form-item">
			<span class="bgr__no-keys" v-if="noKeys">{{ $t('bgr-no-keys-detected') }}</span>

			<table class="bgr__table" v-else>
				<tr>
					<th>Result</th>
					<th>Games</th>
					<th>Key</th>
				</tr>

				<tr v-for="(result, key) in keys" class="key" :class="{ 'key--success': result.Result === 1 }">
					<td>{{ result.PurchaseResultDetail }}</td>
					<td>{{ result.Items ? Object.values(result.Items).join(', ') : '-' }}</td>
					<td>{{ key }}</td>
				</tr>
			</table>
		</div>

		<div class="form-item">
			<div class="form-item__buttons form-item__buttons--center">
				<button class="button button--confirm" @click="$emit('back')" :key="'back'">{{ $t('back') }}</button>
			</div>
		</div>
	</div>
</template>

<script>

	export default {
		props: {
			keys: Object,
			title: String
		},
		name: 'redeem-summary',
		computed: {
			noKeys() {
				return Object.keys(this.keys).length === 0;
			}
		}
	};
</script>

<style lang="scss">
	.bgr__summary-title {
		margin-bottom: 0.5em;
	}
</style>
