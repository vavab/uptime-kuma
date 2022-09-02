<template>
    <!-- Group List -->
    <Draggable
        v-model="$root.publicGroupList"
        :disabled="!editMode"
        item-key="id"
        :animation="100"
    >
        <template #item="group">
            <div class="mb-5 ">
                <!-- Group Title -->
                <h2 class="group-title">
                    <font-awesome-icon v-if="editMode && showGroupDrag" icon="arrows-alt-v" class="action drag me-3" />
                    <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeGroup(group.index)" />
                    <Editable v-model="group.element.name" :contenteditable="editMode" tag="span" />
                </h2>

                <div class="shadow-box monitor-list mt-4 position-relative">
                    <div v-if="group.element.monitorList.length === 0" class="text-center no-monitor-msg">
                        {{ $t("No Monitors") }}
                    </div>

                    <!-- Monitor List -->
                    <!-- animation is not working, no idea why -->
                    <Draggable
                        v-model="group.element.monitorList"
                        class="monitor-list"
                        group="same-group"
                        :disabled="!editMode"
                        :animation="100"
                        item-key="id"
                    >
                        <template #item="monitor">
                            <a class="item">
                                <a class="openlink" href="#">
                                    <div class="row" @click.prevent="monitor.element.showChart = !monitor.element.showChart">
                                        <div class="col-8 col-md-7 small-padding">
                                            <div class="info monitor-title">
                                                <font-awesome-icon v-if="editMode" icon="arrows-alt-v" class="action drag me-3" />
                                                <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeMonitor(group.index, monitor.index)" />
                                                <Uptime :monitor="monitor.element" type="24" :pill="true" />
                                                    {{ monitor.element.name }}
                                            </div>
                                        </div>
                                        <div :key="$root.userHeartbeatBar" class="col-3 col-md-4">
                                            <HeartbeatBar size="small" :monitor-id="monitor.element.id" />
                                        </div>
                                        <div class="col-1 col-md-1">
                                            <div class="info">
                                                <font-awesome-icon v-if="!monitor.element.showChart" icon="sort-down" class="sort" />
                                                <font-awesome-icon v-if="monitor.element.showChart" icon="sort-up" class="sort" />
                                            </div>
                                        </div>
                                    </div>
                                </a>
                                <div v-if="monitor.element.showChart" class="row">
                                    <div v-if="showTags" class="tags">
                                        <Tag v-for="tag in monitor.element.tags" :key="tag" :item="tag" :size="'sm'" />
                                    </div>
                                    <div class="box big-padding text-center monitor-stats">
                                        <div class="row">
                                            <div class="col">
                                                <h5>{{ $t("Temps de réponse") }}</h5>
                                                <p>({{ $t("Actuel") }})</p>
                                                <CountUp :value="publicLastHeartbeatList[monitor.element.id].ping" />
                                            </div>
                                            <div class="col">
                                                <h5>{{ $t("Temps de réponse") }}</h5>
                                                <p>({{ $t("Moyen") }})</p>
                                                <CountUp :value="publicAvgPingList[monitor.element.id]" />
                                            </div>
                                            <div class="col">
                                                <h5>{{ $t("Disponibilité") }}</h5>
                                                <p>({{ $t("24 heures") }})</p>
                                                <span class="num"><Uptime :monitor="monitor.element" type="24" /></span>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="box big-padding text-center ping-chart-wrapper">
                                        <div class="row">
                                            <div class="col">
                                                <PingChart :monitor-id="monitor.element.id" />
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </a>
                        </template>
                    </Draggable>
                </div>
            </div>
        </template>
    </Draggable>
</template>

<script>
import { defineAsyncComponent } from "vue";
import Draggable from "vuedraggable";
import HeartbeatBar from "./HeartbeatBar.vue";
import Uptime from "./Uptime.vue";
import Tag from "./Tag.vue";
const PingChart = defineAsyncComponent(() => import("./PingChart.vue"));
import CountUp from "../components/CountUp.vue";


export default {
    components: {
        Draggable,
        HeartbeatBar,
        Uptime,
        Tag,
        PingChart,
        CountUp,
    },
    props: {
        /** Are we in edit mode? */
        editMode: {
            type: Boolean,
            required: true,
        },
        /** Should tags be shown? */
        showTags: {
            type: Boolean,
        }
    },
    data() {
        return {
            showPingChartBox: false,
        };
    },
    computed: {
        showGroupDrag() {
            return (this.$root.publicGroupList.length >= 2);
        },

        publicLastHeartbeatList() {
            return this.$root.publicLastHeartbeatList;
        },

        publicAvgPingList() {
            let list = {};
            let sum = 0;
            let len = 0;
            for(let monitorId in this.$root.heartbeatList) {
                if(this.$root.heartbeatList[monitorId]) {
                    for(let object in this.$root.heartbeatList[monitorId]) {
                        sum = sum + this.$root.heartbeatList[monitorId][object].ping;
                        len = len + 1;
                    }
                    list[monitorId] = Math.round(sum/len);
                }
            }
            return list;
        },
    },
    created() {

    },
    methods: {
        /**
         * Remove the specified group
         * @param {number} index Index of group to remove
         */
        removeGroup(index) {
            this.$root.publicGroupList.splice(index, 1);
        },

        /**
         * Remove a monitor from a group
         * @param {number} groupIndex Index of group to remove monitor
         * from
         * @param {number} index Index of monitor to remove
         */
        removeMonitor(groupIndex, index) {
            this.$root.publicGroupList[groupIndex].monitorList.splice(index, 1);
        },
    }
};
</script>

<style lang="scss" scoped>
@import "../assets/vars";

.no-monitor-msg {
    position: absolute;
    width: 100%;
    top: 20px;
    left: 0;
}

.monitor-list {
    min-height: 46px;
}

.flip-list-move {
    transition: transform 0.5s;
}

.no-move {
    transition: transform 0s;
}

.drag {
    color: #bbb;
    cursor: grab;
}

.remove {
    color: $danger;
}

.group-title {
    span {
        display: inline-block;
        min-width: 15px;
    }
}

.mobile {
    .item {
        padding: 13px 0 10px;
    }
}

.sort {
    margin: 0 auto;
    width: 100px;
}

.openlink {
    text-decoration: none;
}

.monitor-title {
    font-size: 20px;
    margin-bottom: 9px;
}

.monitor-stats p {
    font-size: 13px;
    color: #666;
}

.monitor-stats {
    padding: 10px;

    .col {
        margin: 20px 0;
    }
}

</style>
