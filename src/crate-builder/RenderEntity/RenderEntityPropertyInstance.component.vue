<template>
    <div>
        <div v-if="configuration.readonly">
            <!-- Just render the value on screen (parse ISO dates for niceness though) -->
            <div v-if="isDateTime(props.data.value)">{{ parseISO(props.data.value) }}</div>
            <div v-else>{{ props.data.value }}</div>
        </div>
        <div v-if="!configuration.readonly">
            <!--  not readonly - try to load the relevant display component-->
            <value-component v-if="isValue()" :definition="props.definition.value" />
            <date-time-component
                v-else-if="isDateTime(props.data.value)"
                :property="props.data.property"
                :value="props.data.value"
                @save:property="savePropertyValue"
            />
            <date-component
                v-else-if="isDate(props.data.value)"
                :property="props.data.property"
                :value="props.data.value"
                @save:property="savePropertyValue"
            />
            <time-component
                v-else-if="isTime(props.data.value)"
                :property="props.data.property"
                :value="props.data.value"
                @save:property="savePropertyValue"
            />
            <number-component
                v-else-if="isNumber(props.data.value)"
                :property="props.data.property"
                :value="props.data.value"
                @save:property="savePropertyValue"
            />
            <select-component
                v-else-if="isSelect()"
                :property="props.data.property"
                :value="props.data.value"
                :definition="props.definition"
                @save:property="savePropertyValue"
            />
            <url-component
                v-else-if="isUrl(props.data.value)"
                :property="props.data.property"
                :value="props.data.value"
                @create:entity="createEntity"
            />
            <text-component
                v-else-if="isText(props.data.value)"
                :type="type"
                :property="props.data.property"
                :value="props.data.value"
                :definition="props.definition"
                @save:property="savePropertyValue"
            />
        </div>
    </div>
</template>

<script setup>
import TextComponent from "../primitives/Text.component.vue";
import DateComponent from "../primitives/Date.component.vue";
import DateTimeComponent from "../primitives/DateTime.component.vue";
import TimeComponent from "../primitives/Time.component.vue";
import NumberComponent from "../primitives/Number.component.vue";
import ValueComponent from "../primitives/Value.component.vue";
import SelectComponent from "../primitives/Select.component.vue";
import UrlComponent from "../primitives/Url.component.vue";
import { parseISO, startOfDay } from "date-fns";
import isString from "lodash/isString";
import { isDate as validatorIsDate, isDecimal, isInt, isFloat, isNumeric } from "validator";
import { computed, inject } from "vue";
import { isURL } from "../crate-manager.js";
const configuration = inject("configuration");

const props = defineProps({
    crateManager: {
        type: Object,
        required: true,
    },
    data: {
        type: Object,
        required: true,
    },
    definition: {
        type: Object,
        required: true,
    },
});
const $emit = defineEmits(["save:property", "create:entity"]);
let type = computed(() => {
    return props?.definition?.type?.[0].toLowerCase();
});
async function savePropertyValue(data) {
    if (!data.propertyId) {
        data = { ...data, property: props.data.property, propertyId: props.data.propertyId };
    }
    $emit("save:property", data);
}
function createEntity(data) {
    $emit("create:entity", data);
}
function isDate(string) {
    const date = parseISO(string);
    return validatorIsDate(date) && definitionIncludes("Date");
}
function isDateTime(string) {
    const date = parseISO(string);
    return validatorIsDate(date) && definitionIncludes("DateTime");
}
function isTime(string) {
    string = string + "";
    return (string.match(/^([0-9]|0[0-9]|1[0-9]|2[0-3]):[0-5][0-9]:[0-5][0-9]$/) ||
        string.match(/^([0-9]|0[0-9]|1[0-9]|2[0-3]):[0-5][0-9]$/)) &&
        (definitionIncludes("DateTime") || definitionIncludes("Time"))
        ? true
        : false;
}
function isText(string) {
    if (
        !isDate(string) &&
        !isDateTime(string) &&
        !isTime(string) &&
        !isNumber(string) &&
        !isUrl(string) &&
        !isSelect() &&
        !isValue()
    )
        return true;
}
function isNumber(string) {
    string = string + "";
    return (
        (isDecimal(string) || isInt(string) || isFloat(string) || isNumeric(string)) &&
        (definitionIncludes("Number") ||
            definitionIncludes("Float") ||
            definitionIncludes("Integer"))
    );
}
function isValue() {
    return props?.definition?.type === "Value";
}
function isSelect() {
    return props?.definition?.values?.includes(props.data.value) ? true : false;
}
function isUrl(string) {
    let result = isURL(string);
    return result;
}

function definitionIncludes(type) {
    return props.definition?.type?.includes(type);
}
</script>
