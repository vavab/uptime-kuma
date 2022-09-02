<template>
    <div
        class="tag-wrapper d-inline-flex"
        :class="{ 'px-3': size == 'normal',
                  'py-1': size == 'normal',
                  'm-2': size == 'normal',
                  'px-2': size == 'sm',
                  'py-0': size == 'sm',
                  'm-1': size == 'sm',
        }"
        :style="{ backgroundColor: item.color, fontSize: size == 'sm' ? '0.7em' : '1em' }"
    >
        <span v-if="!isLink" class="tag-text">{{ displayText }}</span>
        <a v-if="isLink" :href="displayText" target="_blank" class="btn btn-documentation">
            <font-awesome-icon class="right-space" icon="file" />
            {{ $t(" Consulter la documentation") }}
        </a>
        <span v-if="remove != null" class="ps-1 btn-remove" @click="remove(item)">
            <font-awesome-icon icon="times" />
        </span>
    </div>
</template>

<script>
export default {
    props: {
        /** Object representing tag */
        item: {
            type: Object,
            required: true,
        },
        /** Function to remove tag */
        remove: {
            type: Function,
            default: null,
        },
        /**
         * Size of tag
         * @values normal, small
         */
        size: {
            type: String,
            default: "normal",
        }
    },
    computed: {
        displayText() {
            if (this.item.value === "") {
                return this.item.name;
            } else {
                return `${this.item.value}`;
            }
        },
        isLink() {
            if (this.item.name === "link") {
                return true;
            } else {
                return false;
            }
        }
    }
};
</script>

<style lang="scss" scoped>
.tag-wrapper {
    color: black;
    opacity: 0.85;
    border-radius: 10px;

    .dark & {
        opacity: 1;
    }
}

.tag-text {
    padding-bottom: 3px !important;
    padding-top: 3px !important;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: normal;
    font-size: 13px;
}

.btn-remove {
    font-size: 0.9em;
    line-height: 24px;
    opacity: 0.3;
}

.btn-remove:hover {
    opacity: 1;
}
</style>
