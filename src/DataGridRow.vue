<template>
    <tr>
        <td v-if="selectable">
            <wwElement
                v-bind="selectCheckbox"
                :states="isSelected ? ['checked'] : []"
                @click="$emit('update:is-selected', !isSelected)"
            ></wwElement>
        </td>
        <template v-for="column in columns" :key="column.id">
            <DataGridCell
                v-if="column.display"
                :ref="el => registerCellRef(column.id, el)"
                :column="column"
                :item="item"
                :edit="column.editable && edit"
                :columns-element="columnsElement"
            >
            </DataGridCell>
        </template>
        <span v-if="!columns || !columns.length" class="message ww-typo-sub-text flex items-center">
            Please define a column
        </span>
        <DataGridActionCell
            v-if="isEditAvailable"
            :id="id"
            :edit-container="editContainer"
            :editing-container="editingContainer"
            :edit="edit"
            @update:edit="$emit('update:edit', $event)"
            @validate="onValidate"
            @delete="onDelete"
            @cancel="onCancel"
        ></DataGridActionCell>
    </tr>
</template>

<script>
import DataGridActionCell from './DataGridActionCell.vue';
import DataGridCell from './DataGridCell.vue';

export default {
    components: { DataGridActionCell, DataGridCell },
    props: {
        id: { type: undefined, required: true },
        item: { type: Object, required: true },
        columns: { type: Array, required: true },
        columnsElement: { type: Object, required: true },
        isEditAvailable: { type: Boolean, default: false },
        editContainer: { type: Object, required: true },
        editingContainer: { type: Object, required: true },
        selectCheckbox: { type: Object, required: true },
        edit: { type: Boolean, default: false },
        selectable: { type: Boolean, default: false },
        isSelected: { type: Boolean, default: false },
    },
    emits: ['update:edit', 'update:row', 'delete:row', 'update:is-selected'],
    data() {
        return {
            cellRefs: {},
        };
    },
    methods: {
        onValidate() {
            const item = _.cloneDeep(this.item);
            Object.values(this.cellRefs).forEach(row => {
                if (row.column && row.column.type !== 'custom') {
                    _.set(item, row.column && row.column.path, _.cloneDeep(row.internalValue));
                }
            });
            this.$emit('update:row', { value: item, id: this.id });
            this.$emit('update:edit', false);
        },
        onDelete() {
            this.$emit('delete:row', { id: this.id, value: this.item });
        },
        onCancel() {
            Object.values(this.cellRefs).forEach(row => {
                row.resetValue();
            });
            this.$emit('update:edit', false);
        },
        // TODO: we need higher vue version to have auto ref array
        registerCellRef(id, el) {
            if (el) {
                this.cellRefs[id] = el;
            }
        },
    },
};
</script>

<style lang="scss" scoped>
/* wwEditor:start */
.message {
    padding: var(--ww-spacing-02);
    color: var(--ww-color-theme-dark-800);
    background-color: var(--ww-color-theme-dark-50);
    border: 1px solid var(--ww-color-theme-dark-100);
}
/* wwEditor:end */
</style>
