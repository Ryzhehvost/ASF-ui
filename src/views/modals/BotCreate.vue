<template>
	<main class="main-container main-container--modal main-container--bot-create">
			<h2 class="title">Create new bot</h2>

			<h3 class="subtitle" v-if="loading"><font-awesome-icon icon="spinner" size="lg" spin></font-awesome-icon></h3>
			<div class="container" v-else>
				<config-editor :fields="fields" :model="model" :categories="categories" @update="onUpdate"></config-editor>

				<div class="form-item">
					<div class="form-item__buttons">
						<button class="button button--confirm" @click="onUpdate">Save</button>
					</div>
				</div>
			</div>
	</main>
</template>

<script>
	import ConfigEditor from '../../components/ConfigEditor.vue';

	import { post } from '../../utils/http';
	import fetchConfigSchema from '../../utils/fetchConfigSchema';

	import { mapGetters } from 'vuex';
	import loadParameterDescriptions from '../../utils/loadParameterDescriptions';

	const categories = [
		{ name: 'Basic', fields: ['Name', 'SteamLogin', 'SteamPassword', 'Enabled', 'IsBotAccount', 'Paused'] },
		{ name: 'Security', fields: ['PasswordFormat', 'UseLoginKeys'] },
		{ name: 'Access', fields: ['SteamUserPermissions', 'SteamParentalPIN'] },
		{ name: 'Community', fields: ['HandleOfflineMessages'] },
		{ name: 'Trade', fields: ['SteamTradeToken', 'TradingPreferences', 'LootableTypes', 'MatchableTypes', 'AcceptGifts', 'DismissInventoryNotifications'] },
		{ name: 'Farming', fields: ['FarmingOrders', 'SendTradePeriod', 'AutoSteamSaleEvent', 'IdlePriorityQueueOnly', 'IdleRefundableGames', 'FarmOffline', 'SendOnFarmingFinished', 'ShutdownOnFarmingFinished'] },
		{ name: 'Customization', fields: ['SteamMasterClanID', 'GamesPlayedWhileIdle', 'CustomGamePlayedWhileFarming', 'CustomGamePlayedWhileIdle'] },
		{ name: 'Performance', fields: ['HoursUntilCardDrops'] }
	];

	export default {
		name: 'bot-create',
		components: { ConfigEditor },
		metaInfo() {
			return {
				title: `Create new bot`
			};
		},
		data() {
			return {
				loading: true,
				fields: [],
				model: {},
				categories
			};
		},
		computed: {
			...mapGetters({ version: 'status/version' })
		},
		async created() {
			await this.loadConfig();
		},
		methods: {
			async loadConfig() {
				const [{ body: fields }, descriptions] = await Promise.all([
					fetchConfigSchema('ArchiSteamFarm.BotConfig'),
					loadParameterDescriptions(this.version)
				]);

				this.model = {};

				this.fields = [
					{ defaultValue: '', param: 'Name', paramName: 'Name', type: 'string' },
					...Object.keys(fields).map(key => ({
						description: descriptions[key],
						...fields[key]
					}))
				];

				this.loading = false;
			},
			async onUpdate() {
				if (!this.model.Name) return;
				await post(`bot/${this.model.Name}`, { BotConfig: this.model });
				await this.$store.dispatch('bots/updateBots');
				this.$parent.close();
			}
		}
	};
</script>

<style lang="scss">
	.main-container--bot-create {
		width: 1000px;
	}
</style>