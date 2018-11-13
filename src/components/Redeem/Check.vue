<template>
	<div class="bgr__check">
		<div class="form-item">
			<bgr-keys :keys="keys"></bgr-keys>
		</div>

		<div class="form-item">
			<div class="form-item__buttons form-item__buttons--center">
				<button class="button button--confirm" @click="$emit('redeem')" v-if="!noKeys" :class="{ 'button--disabled': confirmingBgr }">
					<font-awesome-icon icon="spinner" v-if="confirmingRedeem" spin></font-awesome-icon>
					<span v-else>{{ $t('redeem') }}</span>
				</button>

				<button class="button button--confirm" @click="$emit('bgr')" v-if="!noKeys" :class="{ 'button--disabled': confirmingRedeem }">
					<font-awesome-icon icon="spinner" v-if="confirmingBgr" spin></font-awesome-icon>
					<span v-else>{{ $t('redeem-with-bgr') }}</span>
				</button>

				<button class="button button--cancel" @click="$emit('cancel')" :key="'cancel'">{{ $t('cancel') }}</button>
			</div>
		</div>
	</div>
</template>

<script>
	import BgrKeys from "./Keys.vue";

	export default {
		components: { BgrKeys },
		props: {
			bot: Object,
			keys: Object,
			confirmingRedeem: Boolean,
			confirmingBgr: Boolean
		},
		name: 'redeem-check',
		computed: {
			noKeys() {
				return !Object.keys(this.keys).length;
			}
		}
	};
</script>
