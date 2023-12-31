<template>
	<n8n-node-icon
		:type="type"
		:src="iconSource.path || iconSource.fileBuffer"
		:name="iconSource.icon"
		:color="color"
		:disabled="disabled"
		:size="size"
		:circle="circle"
		:nodeTypeName="nodeType ? nodeType.displayName : ''"
		:showTooltip="showTooltip"
		@click="(e) => $emit('click')"
	></n8n-node-icon>
</template>

<script lang="ts">
import type { IVersionNode } from '@/Interface';
import { useRootStore } from '@/stores/n8nRoot.store';
import type { INodeTypeDescription } from 'n8n-workflow';
import { mapStores } from 'pinia';
import { defineComponent } from 'vue';

interface NodeIconSource {
	path?: string;
	fileBuffer?: string;
	icon?: string;
}

export default defineComponent({
	name: 'NodeIcon',
	props: {
		nodeType: {},
		size: {
			type: Number,
			required: false,
		},
		disabled: {
			type: Boolean,
			default: false,
		},
		circle: {
			type: Boolean,
			default: false,
		},
		colorDefault: {
			type: String,
			required: false,
		},
		showTooltip: {
			type: Boolean,
			default: false,
		},
	},
	computed: {
		...mapStores(useRootStore),
		type(): string {
			const nodeType = this.nodeType as INodeTypeDescription | IVersionNode | null;
			let iconType = 'unknown';
			if (nodeType) {
				if (nodeType.iconUrl) return 'file';
				if ((nodeType as IVersionNode).iconData) {
					iconType = (nodeType as IVersionNode).iconData.type;
				} else if (nodeType.icon) {
					iconType = nodeType.icon.split(':')[0] === 'file' ? 'file' : 'icon';
				}
			}
			return iconType;
		},
		color(): string {
			const nodeType = this.nodeType as INodeTypeDescription | IVersionNode | null;
			if (nodeType && nodeType.defaults && nodeType.defaults.color) {
				return nodeType.defaults.color.toString();
			}
			if (this.colorDefault) {
				return this.colorDefault;
			}
			return '';
		},
		iconSource(): NodeIconSource {
			const nodeType = this.nodeType as INodeTypeDescription | IVersionNode | null;
			const baseUrl = this.rootStore.getBaseUrl;
			const iconSource = {} as NodeIconSource;

			if (nodeType) {
				// If node type has icon data, use it
				if ((nodeType as IVersionNode).iconData) {
					return {
						icon: (nodeType as IVersionNode).iconData.icon,
						fileBuffer: (nodeType as IVersionNode).iconData.fileBuffer,
					};
				}
				if (nodeType.iconUrl) {
					return { path: baseUrl + nodeType.iconUrl };
				}
				// Otherwise, extract it from icon prop
				if (nodeType.icon) {
					const [type, path] = nodeType.icon.split(':');
					if (type === 'file') {
						throw new Error(`Unexpected icon: ${nodeType.icon}`);
					} else {
						iconSource.icon = path;
					}
				}
			}
			return iconSource;
		},
	},
});
</script>

<style lang="scss"></style>
