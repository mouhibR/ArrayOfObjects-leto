<template>
  <div>
    <div
      v-for="(item, index) in localValue"
      :key="index"
    >
      <q-expansion-item
        v-model="expandedItems[index]"
        expand-icon-class="text-white"
        bordered
        class="rounded-borders shadow-1 col"
        header-class="bg-primary text-white"
        dense
      >
        <template #header>
          <q-item-section avatar>
            <q-icon
              color="accent"
              name="fa-solid fa-lock-open"
              title="Unreferenced"
              size="xs"
            />
          </q-item-section>
          <q-item-section>
            {{ Object.name}} [{{index + 1}}]
          </q-item-section>
          <q-item-section side>
            <q-btn
              flat
              round
              color="negative"
              icon="fa-solid fa-trash"
              @click="removeObject(index)"
            >
              <q-tooltip
                anchor="center left"
                self="center right"
              >
                Remove Object
              </q-tooltip>
            </q-btn>
          </q-item-section>
        </template>
        <div class="q-pa-md">
          <attributes-list
      :attributes="getSubAttributes(attribute)"
      :plugin="plugin"
      :full-name="fullName"
      :current-error="currentError"
      @update:attributes="updateObject(index, $event)"
    />
        </div>
      </q-expansion-item>
    </div>
    <q-btn
      flat
      round
      color="primary"
      icon="fa-solid fa-plus"
      class="q-mt-md"
      @click="addObject"
    >
      Add an Object
    </q-btn>
  </div>
</template>

<script setup>
import {
  ref, defineProps, defineEmits, watch,
} from 'vue';
import AttributesList from 'src/components/inputs/AttributesList'; // Corrected import statement.
import { ComponentAttribute } from 'leto-modelizer-plugin-core';

const props = defineProps({
  attribute: {
    type: Object,
    required: true,
  },
  plugin: {
    type: Object,
    required: true,
  },
  fullName: {
    type: String,
    required: true,
  },
  currentError: {
    type: String,
    default: null,
  },
});

const expandedItems = ref([]);
const localValue = ref(props.attribute.value || []);
console.log(localValue.value);

function getSubAttributes(attribute) {
  const attributes = [];
  const attributeValue = attribute.value || [];

  attribute.definition?.definedAttributes
    ?.forEach((definition) => {
      const attr = attributeValue.find(({ name }) => name === definition.name);

      if (attr) {
        attributes.push(attr);
      } else {
        attributes.push(new ComponentAttribute({
          name: definition.name,
          type: definition.type,
          definition,
        }));
      }
    });

  return [
    ...attributes,
    ...attributeValue.filter(({ definition }) => !definition),
  ];
}

watch(
  () => props.attribute.value,
  (newValue) => {
    localValue.value = newValue || [];
  },
);

const emit = defineEmits([
  'update:attribute-value',
]);

function emitUpdatedValue() {
  emit('update:attribute-value', localValue);
}

function addObject() {
  expandedItems.value.push(true);
  const newObj = [];
  localValue.value.push(newObj);
  emitUpdatedValue();
}

function removeObject(index) {
  expandedItems.value.splice(index, 1);
  localValue.value.splice(index, 1);
  emitUpdatedValue();
}

function updateObject(index, updatedAttributes) {
  localValue[index] = updatedAttributes;
  emitUpdatedValue();
}
</script>
