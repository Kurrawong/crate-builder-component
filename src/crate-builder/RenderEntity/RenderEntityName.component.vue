<template>
    <div class="flex flex-row">
        <div class="w-1/3 xl:w-1/5 flex flex-col">
            <div>Name</div>
            <div class="text-gray-600 font-light text-xs pr-1">
                A short, descriptive name for this item.
            </div>
        </div>
        <div class="w-2/3 xl:w-4/5 flex flex-col" v-if="!configuration.readonly">
            <div class="flex-grow">
                <text-component
                    class="w-full"
                    type="text"
                    :property="data.property"
                    :value="entity.name"
                    @save:property="save"
                />
            </div>
        </div>
        <div class="w-2/3 xl:w-4/5 flex flex-row" v-else>
            {{ entity.name }}
        </div>
    </div>
</template>

<script setup>
import TextComponent from "../primitives/Text.component.vue";
import { reactive, watch, inject } from "vue";
const configuration = inject("configuration");

let props = defineProps({
    entity: {
        type: Object,
        required: true,
    },
});
const emit = defineEmits(["update:entity"]);

let data = reactive({
    property: "name",
    value: props.entity.value,
});
watch(
    () => props.entity.value,
    () => {
        data.value = props.entity.value;
    }
);

async function save(data) {
    emit("update:entity", data);
}
</script>
