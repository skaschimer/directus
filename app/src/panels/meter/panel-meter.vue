<script setup lang="ts">
import { BaseConditionalFillOperators, PanelFunction } from '@/types/panels';
import { computed, unref } from 'vue';
import { useI18n } from 'vue-i18n';

type ConditionalFillFormat = {
	operator: BaseConditionalFillOperators;
	color: string;
	value: number;
};

interface Props {
	showHeader: boolean;
	collection: string;
	field: string;
	width: number;
	height: number;
	fn: PanelFunction;
	data?: { [fn: string]: { [field: string]: number } }[];
	size?: 'full' | 'half';
	decimals?: number;
	strokeWidth?: number;
	roundedStroke?: boolean;
	showPercentage?: boolean;
	color?: string | null;
	max?: number;
	conditionalFill?: ConditionalFillFormat[] | null;
}

const props = withDefaults(defineProps<Props>(), {
	decimals: 0,
	data: () => [],
	strokeWidth: 20,
	roundedStroke: false,
	showPercentage: true,
	color: 'var(--theme--primary)',
	max: 100,
	conditionalFill: () => [],
	size: 'full',
});

const { n } = useI18n();

const percent = computed(() => {
	if (props.data.length === 0) return 0;
	if (!props.fn || !props.field) return 0;
	return Number(props.data[0][props.fn][props.field]) / Number(props.max);
});

const displayValue = computed(() => {
	const percentDecimal = Number((unref(percent) * 100).toFixed(1));
	return props.showPercentage ? n(percentDecimal) + '%' : '';
});

const radius = computed(() => {
	// 24 is 12*2 padding, 4 is 2*2 border
	const widthPx = props.width * 20 - 24 - 4;
	let heightPx = props.height * 20 - 24 - 4;

	// Adjust for header if enabled, v-workspace-tile header has a fixed height of 42px
	if (props.showHeader) heightPx = heightPx - 42;

	const strokeOffset = props.strokeWidth / 2;

	// Determine the shorter dimension
	const minDimension = Math.min(widthPx, heightPx);

	if (props.size == 'half' && heightPx < widthPx) {
		// Added so half circles that have a shorter height than width can maximize the space up to the padding of the height
		return Math.min(widthPx / 2 - strokeOffset, (props.height * 30) / 2 - 12);
	}

	// The radius is half of the shorter dimension inclusive of padding and border
	return minDimension / 2 - strokeOffset;
});

const svgSize = computed(() => {
	const width = unref(radius) * 2 + props.strokeWidth;

	if (props.size === 'half') {
		return {
			width,
			height: unref(radius) + props.strokeWidth,
		};
	}

	return {
		width,
		height: unref(radius) * 2 + props.strokeWidth,
	};
});

const circumference = computed(() => {
	return unref(radius) * 2 * Math.PI;
});

const dashOffset = computed(() => {
	const full = unref(circumference) - unref(percent) * unref(circumference);
	if (props.size === 'half') return full / 2;
	return full;
});

const conditionalColor = computed(() => {
	const defaultColor = props.color ?? 'var(--theme--primary)';

	if (!unref(percent) || !props.conditionalFill?.length) {
		return defaultColor;
	}

	return (
		props.conditionalFill.find((format) => {
			const percentValue = unref(percent) * 100;
			const compareValue = +(format.value ?? 0);

			switch (format.operator || '>=') {
				case '=':
					return percentValue === compareValue;
				case '!=':
					return percentValue !== compareValue;
				case '>':
					return percentValue > compareValue;
				case '>=':
					return percentValue >= compareValue;
				case '<':
					return percentValue < compareValue;
				case '<=':
					return percentValue < compareValue;
			}
		})?.color ?? defaultColor
	);
});

const fontSize = computed(() => Math.min(Math.ceil(unref(circumference) / 100) * 7, 48) + 'px');

const halfSizeOutputOffset = computed(() => unref(radius) / 4 + props.strokeWidth / 2 + 'px');
</script>

<template>
	<div class="panel-meter" :class="[{ 'has-header': showHeader }, `size-${size}`]">
		<output>{{ displayValue }}</output>
		<svg :width="svgSize.width" :height="svgSize.height">
			<circle
				cx="50%"
				:cy="size === 'half' ? '100%' : '50%'"
				fill="none"
				:stroke-width="strokeWidth"
				:stroke-linecap="roundedStroke ? 'round' : 'inherit'"
				:r="radius"
				stroke="var(--theme--background-subdued)"
			/>
			<circle
				cx="50%"
				:cy="size === 'half' ? '100%' : '50%'"
				fill="none"
				:stroke-width="strokeWidth"
				:stroke-linecap="roundedStroke ? 'round' : 'inherit'"
				:r="radius"
				:stroke="conditionalColor"
			/>
		</svg>
	</div>
</template>

<style scoped>
.panel-meter {
	inline-size: 100%;
	block-size: 100%;
	position: relative;
	padding: 12px;
	display: grid;
	place-items: center center;
	justify-content: center;
}

.panel-meter > * {
	grid-column: 1;
	grid-row: 1;
}

.panel-meter.has-header {
	padding-block-start: 0;
}

.panel-meter output {
	z-index: 1;
	font-weight: 800;
	font-size: v-bind(fontSize);
	line-height: 52px;
}

.panel-meter.size-half output {
	position: relative;
	inset-block-start: v-bind(halfSizeOutputOffset);
}

.panel-meter svg circle:last-child {
	transform-origin: center center;
	transform: rotate(-90deg);
	stroke-dasharray: v-bind(circumference) v-bind(circumference);
	stroke-dashoffset: v-bind(dashOffset);
}

.panel-meter.size-half svg circle:last-child {
	transform-origin: bottom;
	transform: rotate(0deg);
}
</style>
