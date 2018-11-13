<template>
	<main class="main-container main-container--modal">
		<h2 class="title" v-if="!bot">Not found!</h2>

		<template v-else>
			<h2 class="title">{{ bot.name }}</h2>

			<h3 class="subtitle" v-if="loading"><font-awesome-icon icon="spinner" size="lg" spin></font-awesome-icon></h3>
			<bgr-status v-if="!loading && state === 'input'" :used-keys="usedKeys" :unused-keys="unusedKeys" @reset="onReset" @show-unused="state = 'unusedKeys'" @show-used="state = 'usedKeys'"></bgr-status>

			<keep-alive>
				<redeem-input v-if="state === 'input'" @check="onCheck"></redeem-input>
				<redeem-check v-if="state === 'check'" :keys="keys" :bot="bot" :confirming-bgr="confirmingBgr" :confirming-redeem="confirmingRedeem" @bgr="onBGR" @redeem="onRedeem" @cancel="onCancel"></redeem-check>
				<redeem-summary v-if="state === 'redeem-summary'" :keys="summaryKeys" :title="$t('redeem-summary')" @back="$parent.close()"></redeem-summary>
				<bgr-summary v-if="state === 'bgr-summary'" :keys="summaryKeys" :title="$t('bgr-summary-success')" @back="$parent.close()"></bgr-summary>
				<bgr-summary v-if="state === 'usedKeys'" :keys="usedKeys" :title="$t('bgr-used-keys')" @back="state = 'input'"></bgr-summary>
				<bgr-summary v-if="state === 'unusedKeys'" :keys="unusedKeys" :title="$t('bgr-unused-keys')" @back="state = 'input'"></bgr-summary>
			</keep-alive>
		</template>
	</main>
</template>

<script>
	import RedeemCheck from '../../components/Redeem/Check.vue';
	import RedeemInput from '../../components/Redeem/Input.vue';
	import BgrStatus from '../../components/Redeem/BgrStatus.vue';
	import BgrSummary from '../../components/Redeem/BgrSummary.vue';
	import RedeemSummary from '../../components/Redeem/Summary.vue';

	export default {
		name: 'bot-redeem',
		components: { BgrStatus, RedeemInput, RedeemCheck, BgrSummary, RedeemSummary },
		data() {
			return {
				loading: true,
				confirmingBgr: false,
				confirmingRedeem: false,
				state: 'input',
				unusedKeys: {},
				usedKeys: {},
				keys: {},
				summaryKeys: {}
			};
		},
		computed: {
			bot() {
				return this.$store.getters['bots/bot'](this.$route.params.bot);
			}
		},
		watch: {
			'$route': {
				immediate: true,
				handler: async function() {
					this.loading = true;
					const { UnusedKeys, UsedKeys } = await this.loadBGR();
					this.unusedKeys = UnusedKeys;
					this.usedKeys = UsedKeys;
					this.loading = false;
				}
			}
		},
		methods: {
			async loadBGR() {
				if (!this.bot) return { UnusedKeys: {}, UsedKeys: {} };
				return (await this.$http.get(`bot/${this.bot.name}/GamesToRedeemInBackground`))[this.bot.name];
			},
			onCheck(keys) {
				this.keys = keys;
				this.state = 'check';
			},
			async onRedeem() {
				if (this.confirmingBgr || this.confirmingRedeem) return;
				this.confirmingRedeem = true;

				try {
					const response = await this.$http.post(`bot/${this.bot.name}/redeem`, { KeysToRedeem: Object.keys(this.keys) });
					this.state = 'redeem-summary';
					this.summaryKeys = response[this.bot.name];
				} finally {
					this.confirmingRedeem = false;
				}
			},
			async onBGR() {
				if (this.confirmingBgr || this.confirmingRedeem) return;
				this.confirmingBgr = true;

				try {
					const activatedKeys = await this.$http.post(`bot/${this.bot.name}/GamesToRedeemInBackground`, { GamesToRedeemInBackground: this.keys });
					this.state = 'bgr-summary';
					this.summaryKeys = activatedKeys[this.bot.name];
				} finally {
					this.confirmingBgr = false;
				}
			},
			onCancel() {
				this.state = 'input';
			},
			async onReset() {
				await this.$http.del(`bot/${this.bot.name}/GamesToRedeemInBackground`);
				this.unusedKeys = {};
				this.usedKeys = {};
			}
		}
	};
</script>
