<template>
    <div class="col-md-12 mb-2 rowItem row" :id="row.name">
        <div class="tools">
            <font-awesome-icon icon="times" class="clickable" @click="removeRow(row.name)"></font-awesome-icon>
        </div>

        <control-item v-for="(control, index) in row.controls"
                      :control="control"
                      :key="control.name"
                      :label-position="labelPosition">
        </control-item>
    </div>
</template>

<script>
    import {FORM_CONSTANTS} from "sethFormBuilder/config/constants";
    import ControlItem from "./ControlItem";
    import {eventBus, EventHandlerConstant} from 'sethFormBuilder/template/handler/event_handler';
    import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
    import {Hooks} from 'sethFormBuilder/template/components/hook_lists';

    export default {
        components: {ControlItem, FontAwesomeIcon},
        name: "row-item",
        props: {
            row: {
                type: Object,
                default: {}
            },
            labelPosition: null
        },
        methods: {
            addControl(controlType) {
                var controlInfo = _.cloneDeep(FORM_CONSTANTS.Control);
                controlInfo.type = controlType;
                // generate id
                controlInfo.name = _.domUniqueID(`control_${controlType}_`);
                controlInfo.label = FORM_CONSTANTS.Type[controlType].label;
                controlInfo.fieldName = controlInfo.name; // same for both

                // before hook
                let b4Run = Hooks.Control.beforeAdd.runSequence(controlInfo, this.row);
                if (b4Run === false) {
                    return;
                }

                // add to row
                this.row.controls.push(controlInfo);

                // after hook
                Hooks.Control.afterAdd.run(controlInfo, this.row);
            },
            traverseControl() {
                let self = this;

                // prepare data
                var items = $(this.$el).find('.controlItem');
                var finalItems = [];

                // sort
                _.each(items, (item, index) => {
                    var id = $(item).attr('id');
                    var controlItem = _.find(self.row.controls, {name: id});
                    controlItem.order = index;
                    finalItems.push(controlItem);
                });

                // reset the current sections
                this.row.controls = finalItems;
            },
            removeRow(rowName) {
                this.$emit('removeRow', rowName);
            }
        },
        created() {
            eventBus.$on(EventHandlerConstant.REMOVE_CONTROL, ui => {
                // prepare data
                var id = ui.helper.attr('data-control-name');
                var controlIndex = _.findIndex(this.row.controls, {name: id});

                if (controlIndex < 0) {
                    return;
                }

                // before hook
                var controlInfo = this.row.controls[controlIndex];
                let beforeRun = Hooks.Control.beforeRemove.runSequence(controlInfo);
                if (beforeRun === false) {
                    return;
                }

                // remove control
                this.row.controls.splice(controlIndex, 1);

                // after hook
                Hooks.Control.afterRemove.run(controlInfo);
            });
        },
        mounted() {
            let self = this;
            $(this.$el).droppable({
                accept: ".control-wrapper",
                drop: function (event, ui){
                    self.addControl($(ui.draggable[0]).attr('data-control-type'));
                },
                over: function( event, ui ) {
                    ui.helper.css('border', '1px solid green');
                },
            });

            $(this.$el).sortable({
                cursor: "move",
                delay: 100,
                helper: 'clone',
                update: function (event, ui) {
                    self.traverseControl();
                }
            }).disableSelection();
        }
    }
</script>

<style scoped>
    .rowItem {
        border-radius: 10px;
        background-color:rgba(0,0,0,.03);
        padding: 30px 10px;
        margin: 0;
        position: relative;
    }

    .rowItem .tools {
        position: absolute;
        top: 3px;
        right: 12px;
    }

    .clickable {
        cursor: pointer;
    }
</style>
