<script setup lang="ts">
import { useAppStore } from '@directus/stores';
import { toRefs } from 'vue';

withDefaults(
	defineProps<{
		to?: string;
		icon?: string;
		active?: boolean;
	}>(),
	{
		icon: 'box',
	},
);

defineEmits<{
	(e: 'click', event: MouseEvent): void;
}>();

const appStore = useAppStore();
const { sidebarOpen } = toRefs(appStore);
</script>

<template>
	<component
		:is="to ? 'router-link' : 'button'"
		:aria-expanded="to ? undefined : active"
		class="sidebar-button"
		:class="{ active }"
		@click="$emit('click', $event)"
	>
		<div class="icon">
			<v-icon :name="icon!" />
		</div>
		<div v-if="sidebarOpen" class="title">
			<slot />
		</div>
	</component>
</template>

<style lang="scss" scoped>
.sidebar-button {
	--focus-ring-offset: var(--focus-ring-offset-inset);

	position: relative;
	flex-shrink: 0;
	width: 100%;
	height: 60px;
	color: var(--theme--foreground-accent);
	background-color: var(--theme--background-accent);

	.icon {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 60px;
		height: 100%;
	}

	.title {
		position: absolute;
		top: 50%;
		left: 52px;
		overflow: hidden;
		white-space: nowrap;
		transform: translateY(-50%);
	}

	&.active {
		background-color: var(--theme--background-accent);
	}
}
</style>
