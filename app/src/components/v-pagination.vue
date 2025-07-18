<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
	/** Currently selected page */
	modelValue: number;
	/** The amount of pages to render */
	length: number;
	/** Disables the pagination */
	disabled?: boolean;
	/** Specify the max total visible pagination numbers */
	totalVisible?: number;
	/** Show first/last buttons */
	showFirstLast?: boolean;
}>();

const emit = defineEmits(['update:modelValue']);

const visiblePages = computed<number[]>(() => {
	if (props.totalVisible === undefined) return [];

	let startPage: number;
	let endPage: number;

	if (!Number.isInteger(props.totalVisible) || props.length <= props.totalVisible) {
		startPage = 1;
		endPage = props.length;
	} else {
		const pagesBeforeCurrentPage = Math.floor(props.totalVisible / 2);
		const pagesAfterCurrentPage = Math.ceil(props.totalVisible / 2) - 1;

		if (props.modelValue <= pagesBeforeCurrentPage) {
			startPage = 1;
			endPage = props.totalVisible;
		} else if (props.modelValue + pagesAfterCurrentPage >= props.length) {
			startPage = props.length - props.totalVisible + 1;
			endPage = props.length;
		} else {
			startPage = props.modelValue - pagesBeforeCurrentPage;
			endPage = props.modelValue + pagesAfterCurrentPage;
		}
	}

	return Array.from(Array(endPage + 1 - startPage).keys()).map((i) => startPage + i);
});

function toPrev() {
	toPage(props.modelValue - 1);
}

function toNext() {
	toPage(props.modelValue + 1);
}

function toPage(page: number) {
	emit('update:modelValue', page);
}
</script>

<template>
	<div class="v-pagination">
		<v-button class="previous" :disabled="disabled || modelValue === 1" secondary icon small @click="toPrev">
			<v-icon name="chevron_left" />
		</v-button>

		<v-button
			v-if="showFirstLast && totalVisible && modelValue > Math.ceil(totalVisible / 2) + 1 && length > totalVisible"
			class="page"
			secondary
			small
			:disabled="disabled"
			@click="toPage(1)"
		>
			1
		</v-button>

		<span
			v-if="showFirstLast && totalVisible && modelValue > Math.ceil(totalVisible / 2) + 1 && length > totalVisible + 1"
			class="gap"
		>
			...
		</span>

		<v-button
			v-for="page in visiblePages"
			:key="page"
			:class="{ active: modelValue === page }"
			class="page"
			secondary
			small
			:disabled="disabled"
			@click="toPage(page)"
		>
			{{ page }}
		</v-button>

		<span
			v-if="
				showFirstLast && totalVisible && modelValue < length - Math.ceil(totalVisible / 2) && length > totalVisible + 1
			"
			class="gap"
		>
			...
		</span>

		<v-button
			v-if="
				showFirstLast && totalVisible && modelValue <= length - Math.ceil(totalVisible / 2) && length > totalVisible
			"
			:class="{ active: modelValue === length }"
			class="page"
			secondary
			small
			:disabled="disabled"
			@click="toPage(length)"
		>
			{{ length }}
		</v-button>

		<v-button class="next" :disabled="disabled || modelValue === length" secondary icon small @click="toNext">
			<v-icon name="chevron_right" />
		</v-button>
	</div>
</template>

<style scoped>
.v-pagination {
	display: flex;
}

.gap {
	display: none;
	margin: 0 4px;
	color: var(--theme--foreground-subdued);
	line-height: 2em;
}

@media (min-width: 600px) {
	.gap {
		display: inline;
	}
}

.v-button {
	--v-button-background-color-hover: var(--theme--background-normal);
	--v-button-background-color: var(--theme--form--field--input--background-subdued);
	--v-button-color: var(--theme--foreground);

	margin: 0 2px;
	vertical-align: middle;
}

.v-button.page:not(.active) {
	display: none;
}

@media (min-width: 600px) {
	.v-button.page:not(.active) {
		display: inline;
	}
}

.v-button :deep(.small) {
	--v-button-min-width: 32px;
}

.v-button:first-child {
	margin-inline-start: 0;
}

.v-button:last-child {
	margin-inline-end: 0;
}

.v-button.active {
	--v-button-background-color-hover: var(--theme--primary);
	--v-button-color-hover: var(--foreground-inverted);
	--v-button-background-color: var(--theme--primary);
	--v-button-color: var(--foreground-inverted);
}
</style>
