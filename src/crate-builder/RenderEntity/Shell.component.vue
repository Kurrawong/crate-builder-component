<template>
    <div class="flex flex-col">
        <!-- <pre>{{ data.entity }}</pre> -->
        <div v-if="!data.tabs.length">
            <!-- render controls -->
            <render-controls-component
                v-if="!configuration.readonly"
                :crate-manager="props.crateManager"
                :entity="data.entity"
                @load:entity="loadEntity"
                @add:property:placeholder="addPropertyPlaceholder"
                @delete:entity="deleteEntity"
                @save:crate:template="saveCrateAsTemplate"
                @save:entity:template="saveEntityAsTemplate"
                @update:context="updateContext"
            />

            <!-- render entity id -->
            <render-entity-id-component
                class="my-2 p-2"
                :class="{
                    'bg-green-200 rounded p-1 my-1': data.savedProperty === '@id',
                }"
                :entity="data.entity"
                @update:entity="updateEntity"
            />

            <!-- render entity type -->
            <render-entity-type-component :entity="data.entity" class="my-2 p-2" />

            <!-- render entity name -->
            <!-- <render-entity-name-component
                class="my-2 p-2"
                :class="{
                    'bg-green-200 rounded p-1 my-1': data.savedProperty === 'name',
                }"
                :entity="data.entity"
                @update:entity="updateEntity"
            /> -->

            <!-- render entity properties -->
            <div v-for="(values, property) of data.entity.properties" :key="property">
                <div
                    v-if="showProperty(property)"
                    :class="{
                        'bg-green-200 rounded p-1 my-1': data.savedProperty === property,
                    }"
                >
                    <render-entity-property-component
                        class="my-2"
                        :crate-manager="props.crateManager"
                        :entity="props.entity"
                        :property="property"
                        :values="values"
                        @load:entity="loadEntity"
                        @create:property="createProperty"
                        @save:property="saveProperty"
                        @delete:property="deleteProperty"
                        @create:entity="createEntity"
                        @link:entity="linkEntity"
                        @add:template="addTemplate"
                    />
                </div>
            </div>

            <!--render entities it links to  -->
            <div class="flex flex-row flex-wrap">
                <div v-for="(entities, property) of data.entity.reverseConnections" :key="property">
                    <div v-for="entity of entities" :key="entity.tgtEntityId">
                        <render-entity-reverse-item-link-component
                            class="m-1"
                            :crate-manager="props.crateManager"
                            :property="property"
                            :entity="entity"
                            @load:entity="loadEntity"
                        />
                    </div>
                </div>
            </div>
        </div>
        <div v-if="data.tabs.length">
            <!-- grouped profile -->
            <div class="flex flex-col flex-grow">
                <!-- render controls -->
                <render-controls-component
                    v-if="!configuration.readonly"
                    :crate-manager="props.crateManager"
                    :entity="data.tabs[0].entity"
                    @load:entity="loadEntity"
                    @add:property:placeholder="addPropertyPlaceholder"
                    @delete:entity="deleteEntity"
                    @save:crate:template="saveCrateAsTemplate"
                    @save:entity:template="saveEntityAsTemplate"
                    @update:context="updateContext"
                />
                <el-tabs tab-position="left" v-model="data.activeTab">
                    <el-tab-pane
                        :label="tab.name"
                        :name="tab.name"
                        v-for="(tab, idx) of data.tabs"
                        :key="idx"
                    >
                        <template #label>
                            <div
                                class="flex flex-col tab-label-width whitespace-normal text-gray-600"
                            >
                                <div
                                    class="cursor-pointer text-lg hover:text-yellow-600"
                                    v-if="tab.label"
                                >
                                    {{ tab.label }}
                                </div>
                                <div
                                    class="cursor-pointer text-lg hover:text-yellow-600"
                                    v-else-if="tab.name"
                                >
                                    {{ tab.name }}
                                </div>
                                <div class="text-gray-600 font-light text-xs pr-1 pb-4">
                                    {{ tab.description }}
                                </div>
                            </div>
                        </template>

                        <span v-if="data.activeTab === tab.name">
                            <div v-if="tab.name === 'About'">
                                <!-- render entity id -->
                                <render-entity-id-component
                                    class="my-2 p-2"
                                    :class="{
                                        'bg-green-200 rounded p-1 my-1':
                                            data.savedProperty === '@id',
                                    }"
                                    :entity="data.tabs[0].entity"
                                    @update:entity="updateEntity"
                                />
                                <!-- render entity type -->
                                <render-entity-type-component
                                    class="my-2 p-2"
                                    :entity="data.tabs[0].entity"
                                />

                                <!-- render entity name -->
                                <!-- <render-entity-name-component
                                    class="my-2 p-2"
                                    :class="{
                                        'bg-green-200 rounded p-1 my-1':
                                            data.savedProperty === 'name',
                                    }"
                                    :entity="data.tabs[0].entity"
                                    @save:property="saveProperty"
                                    @update:entity="updateEntity"
                                /> -->
                            </div>

                            <!-- render entity properties -->
                            <div
                                v-for="(values, property) of tab.entity.properties"
                                :key="property"
                            >
                                <div
                                    v-if="showProperty(property)"
                                    :class="{
                                        'bg-green-200 rounded p-1 my-1':
                                            data.savedProperty === property,
                                    }"
                                >
                                    <render-entity-property-component
                                        class="my-2"
                                        :crate-manager="props.crateManager"
                                        :entity="tab.entity"
                                        :property="property"
                                        :values="values"
                                        @load:entity="loadEntity"
                                        @create:property="createProperty"
                                        @save:property="saveProperty"
                                        @delete:property="deleteProperty"
                                        @create:entity="createEntity"
                                        @link:entity="linkEntity"
                                        @add:template="addTemplate"
                                    />
                                </div>
                            </div>

                            <div v-if="tab.name === 'About'">
                                <!--render entities it links to  -->
                                <div class="flex flex-row flex-wrap">
                                    <div
                                        v-for="(entities, property) of data.tabs[0].entity
                                            .reverseConnections"
                                        :key="property"
                                    >
                                        <div v-for="entity of entities" :key="entity.tgtEntityId">
                                            <render-entity-reverse-item-link-component
                                                class="m-2"
                                                :crate-manager="props.crateManager"
                                                :property="property"
                                                :entity="entity"
                                                @load:entity="loadEntity"
                                            />
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </span>
                    </el-tab-pane>
                </el-tabs>
            </div>
        </div>
    </div>
</template>

<script setup>
import RenderEntityIdComponent from "./RenderEntityId.component.vue";
import RenderEntityTypeComponent from "./RenderEntityType.component.vue";
// import RenderEntityNameComponent from "./RenderEntityName.component.vue";
import RenderEntityPropertyComponent from "./RenderEntityProperty.component.vue";
import RenderEntityReverseItemLinkComponent from "./RenderReverseItemLink.component.vue";
import RenderControlsComponent from "./RenderControls.component.vue";
import { reactive, onMounted, onBeforeMount, watch, provide } from "vue";
import { debounce, cloneDeep } from "lodash";
import { ProfileManager } from "../profile-manager.js";

const props = defineProps({
    crateManager: {
        type: Object,
        required: true,
    },
    entity: {
        type: Object,
        required: true,
    },
    configuration: {
        type: Object,
        required: true,
    },
});

const data = reactive({
    hideProperty: ["describoId"],
    classDefinition: undefined,
    entity: {},
    activeTab: "About",
    tabs: [],
    debouncedInit: debounce(init, 400),
    extraProperties: [],
    savedProperty: undefined,
    savedPropertyTimeout: 1000,
});

const $emit = defineEmits([
    "ready",
    "load:entity",
    "save:crate",
    "save:crate:template",
    "save:entity:template",
    "add:property",
    "save:property",
    "delete:property",
    "ingest:entity",
    "link:entity",
    "update:entity",
    "delete:entity",
]);

watch(
    () => props.entity,
    (n, o) => {
        if (n.describoId !== o.describoId) {
            data.extraProperties = [];
            data.entity = {};
            data.tabs = [];
        }
        data.debouncedInit();
    }
);
onBeforeMount(() => {
    provide("configuration", props.configuration);
});
onMounted(() => {
    data.debouncedInit();
});

function init() {
    if (!props.entity.describoId) return;
    const profileManager = new ProfileManager({ profile: props.crateManager.profile });

    let entity;
    if (props.configuration.mode === "embedded") {
        entity = {
            ...props.entity,
            properties: props.crateManager.getEntityProperties({
                describoId: props.entity.describoId,
                grouped: true,
            }),
            reverseConnections: props.crateManager.getEntityReverseConnections({
                describoId: props.entity.describoId,
                grouped: true,
            }),
        };
    } else if (props.configuration.mode === "online") {
        entity = props.entity;
    }

    const typeDefinition = profileManager.getTypeDefinition({ entity });

    typeDefinition?.inputs.forEach((input) => {
        if (input.name === "name") return;
        if (entity.properties[input.name]) {
            entity.properties[input.name] = entity.properties[input.name];
        } else {
            if (!props.configuration.readonly) entity.properties[input.name] = [];
        }
    });
    if (data.extraProperties.length) {
        data.extraProperties.forEach((property) => {
            if (!entity.properties[property]) entity.properties[property] = [];
        });
    }
    let properties = {};
    let propertyNames = [];
    Object.keys(entity.properties).map((k) => (propertyNames[k.toLowerCase()] = k));
    Object.keys(entity.properties)
        .map((k) => k.toLowerCase())
        .sort()
        .forEach((k) => (properties[propertyNames[k]] = entity.properties[propertyNames[k]]));
    entity.properties = properties;

    const { layouts, hide } = profileManager.getLayout({ type: entity["@type"] });
    let layout = applyLayout({ layouts, hide, entity });
    if (layout.entity) {
        data.entity = { ...entity, ...layout.entity };
    } else if (layout.tabs) {
        data.tabs = cloneDeep(layout.tabs);
    }
    $emit("ready");
}
function applyLayout({ layouts, hide = [], entity }) {
    if (!layouts?.length) return { entity };

    let tabs = [];

    let mappedInputs = [];
    layouts.forEach((section) => {
        let sectionEntity = cloneDeep(entity);
        sectionEntity.properties = {};

        section.inputs.forEach((input) => {
            let property = Object.keys(entity.properties).filter((property) => property === input);
            if (property.length && !hide.includes(input)) {
                mappedInputs.push(input);
                sectionEntity.properties[input] = entity.properties[input];
            }
        });
        tabs.push({
            ...section,
            entity: sectionEntity,
        });
    });

    let unmappedInputs = Object.keys(entity.properties).filter((p) => !mappedInputs.includes(p));
    if (unmappedInputs.length) {
        let sectionEntity = cloneDeep(entity);
        sectionEntity.properties = {};
        unmappedInputs.forEach((p) => {
            if (!hide.includes(p)) {
                sectionEntity.properties[p] = entity.properties[p];
            }
        });
        // is there an ungrouped properties tab?
        let ungroupedTab = tabs.filter((tab) => tab.name.match(/\.\.\./));
        if (ungroupedTab.length) {
            tabs = tabs.map((tab) => {
                if (tab.name === "...") {
                    return {
                        ...tab,
                        entity: sectionEntity,
                    };
                }
                return tab;
            });
        } else {
            tabs.push({ name: "...", description: "", entity: sectionEntity });
        }
    }

    // is there an about tab?
    const aboutTab = tabs.filter((tab) => tab.name.match(/about/i));
    if (!aboutTab.length) {
        let sectionEntity = cloneDeep(entity);
        delete sectionEntity.properties;
        tabs = [{ name: "About", entity: sectionEntity }, ...tabs];
    }

    return { tabs };
}
function addPropertyPlaceholder({ property }) {
    data.extraProperties.push(property);
    init();
}
function showProperty(property) {
    return !data.hideProperty.includes(property);
}
function loadEntity(entity) {
    if (props.configuration.mode === "embedded") {
        if (props.crateManager.getEntity({ describoId: props.crateManager.currentEntity })) {
            $emit("load:entity", { describoId: props.crateManager.currentEntity });
        }
    }
    $emit("load:entity", entity);
}
function saveCrate() {
    $emit("save:crate");
}
function createProperty(patch) {
    console.debug("Render Entity component: emit(create:property)", patch);
    if (props.configuration.mode === "embedded") {
        props.crateManager.addProperty({ ...patch });
    } else {
        $emit("add:property", { ...patch, entityId: props.entity.describoId });
    }
    data.savedProperty = patch.property;
    setTimeout(() => (data.savedProperty = undefined), data.savedPropertyTimeout);
    saveCrate();
    init();
}
function saveProperty(patch) {
    console.debug("Render Entity component: emit(save:property)", patch);
    if (props.configuration.mode === "embedded") {
        props.crateManager.updateProperty({ ...patch });
    } else {
        $emit("save:property", patch);
    }
    data.savedProperty = patch.property;
    setTimeout(() => (data.savedProperty = undefined), data.savedPropertyTimeout);
    saveCrate();
    init();
}
function deleteProperty(data) {
    console.debug("Render Entity component: emit(delete:property)", data);
    if (props.configuration.mode === "embedded") {
        props.crateManager.deleteProperty({ propertyId: data.propertyId });
    } else {
        $emit("delete:property", { propertyId: data.propertyId });
    }
    saveCrate();
    init();
}
function createEntity(data) {
    const property = data.property;
    delete data.property;
    const dataType = data.json.type;
    delete data.json.type;
    console.debug("Render Entity component: emit(create:entity)", data);
    if (props.configuration.mode === "embedded") {
        if (dataType === "datapack") {
            props.crateManager.ingestAndLink({ property, json: data });
        } else {
            let entity = props.crateManager.addEntity({ entity: data.json });
            props.crateManager.linkEntity({
                srcEntityId: props.entity.describoId,
                property,
                tgtEntityId: entity.describoId,
            });
        }
    } else {
        $emit("ingest:entity", { property, entityId: props.entity.describoId, json: data });
    }
    init();
    saveCrate();
}
function updateEntity(patch) {
    console.debug("Render Entity component: emit(update:entity)", patch);
    if (props.configuration.mode === "embedded") {
        props.crateManager.updateEntity({ ...patch });
    } else {
        $emit("update:entity", { ...patch, entityId: props.entity.describoId });
    }
    data.savedProperty = patch.property;
    setTimeout(() => (data.savedProperty = undefined), data.savedPropertyTimeout);
    init();
    saveCrate();
}
function linkEntity(data) {
    console.debug("Render Entity component: emit(link:entity)", data);
    if (props.configuration.mode === "embedded") {
        props.crateManager.linkEntity({
            property: data.property,
            tgtEntityId: data.json.describoId,
        });
    } else {
        $emit("link:entity", { property: data.property, tgtEntityId: data.describoId });
    }
    init();
    saveCrate();
}
function deleteEntity(data) {
    console.debug("Render Entity component: emit(delete:entity)", data);
    if (props.configuration.mode === "embedded") {
        props.crateManager.deleteEntity(data);
        init();
    } else {
        $emit("delete:entity", data);
    }
    saveCrate();
}
function saveCrateAsTemplate(data) {
    console.debug("Render Entity component: emit(save:crate:template)", data);
    $emit("save:crate:template", data);
}
function saveEntityAsTemplate() {
    console.debug("Render Entity component: emit(save:entity:template)");
    $emit("save:entity:template");
}
function updateContext(data) {
    props.crateManager.context = data;
    saveCrate();
}
function addTemplate() {}
</script>

<style scoped>
.tab-label-width {
    width: 200px;
}
</style>
