<template>
  <slot></slot>
</template>

<script setup lang="ts">
import {
  provide,
  inject,
  watch,
  onMounted,
  onUnmounted,
  shallowRef,
} from "vue";
import Transform, {
  type RotateEvent,
  type ScaleEvent,
  type SelectEvent,
  type TranslateEvent,
  type Options,
} from "ol-ext/interaction/Transform";
import type Map from "ol/Map";
import usePropsAsObjectProperties from "@/composables/usePropsAsObjectProperties";
import {
  useOpenLayersEvents,
  type CommonEvents,
} from "@/composables/useOpenLayersEvents";
import { always } from "ol/events/condition";
import { TRUE } from "ol/functions";

const props = withDefaults(defineProps<Options>(), {
  translateFeature: true,
  scale: true,
  rotate: true,
  keepAspectRatio: always,
  translate: true,
  stretch: true,
  selection: TRUE,
});

type Emits = CommonEvents & {
  (e: "select", event: SelectEvent): void;
  (e: "rotatestart", event: RotateEvent): void;
  (e: "rotating", event: RotateEvent): void;
  (e: "rotateend", event: RotateEvent): void;
  (e: "translatestart", event: TranslateEvent): void;
  (e: "translating", event: TranslateEvent): void;
  (e: "translateend", event: TranslateEvent): void;
  (e: "scalestart", event: ScaleEvent): void;
  (e: "scaling", event: ScaleEvent): void;
  (e: "scaleend", event: ScaleEvent): void;
};
defineEmits<Emits>();

// prevent warnings caused by event pass-through via useOpenLayersEvents composable
defineOptions({
  inheritAttrs: false,
});

const map = inject<Map>("map");

const properties = usePropsAsObjectProperties(props);

function createTransform() {
  return new Transform(properties as Options);
}

const transform = shallowRef(createTransform());

const { updateOpenLayersEventHandlers } = useOpenLayersEvents(transform, [
  "select",
  "rotatestart",
  "rotating",
  "rotateend",
  "translatestart",
  "translating",
  "translateend",
  "scalestart",
  "scaling",
  "scaleend",
]);

watch(transform, (newVal, oldVal) => {
  map?.removeInteraction(oldVal);
  transform.value = createTransform();
  updateOpenLayersEventHandlers();
  map?.addInteraction(newVal);
  map?.changed();
});

onMounted(() => {
  map?.addInteraction(transform.value);
});

onUnmounted(() => {
  map?.removeInteraction(transform.value);
});

provide("stylable", transform);

defineExpose({
  transform: transform.value,
});
</script>
