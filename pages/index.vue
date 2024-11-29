<template>
    <div>
        <h1 class="font-bold text-2xl">New Shipment</h1>

        <div class="bg-white shadow-lg rounded-lg p-5 mt-5">
            <h2 class="font-semibold">Shipment Data</h2>

            <!-- Shipment Reference Number -->
            <div class="mt-4 flex flex-col gap-2">
                <div class="flex gap-2">
                    <label class="font-medium uppercase text-xs text-gray-700">Shipment Reference Number</label>
                    <span class="text-red-600 relative -top-1" v-if="r$.$fields.referenceNumber.$isRequired">*</span>
                </div>

                <input v-model="form.referenceNumber" type="text" class="w-1/2 ring-1 ring-gray-300 rounded outline-none p-2" />

                <FieldError :errors="r$.$errors.referenceNumber" />
            </div>

            <!-- Shipment Items -->
            <h3 class="font-semibold my-4">Shipment Items</h3>

            <div v-for="(_, index) in form.shipmentItems" class="border border-dashed border-gray-300 p-4 rounded-lg mt-4">
                <h3 class="font-semibold uppercase text-xs text-gray-700 mb-3">
                    Item {{ index + 1 }}
                </h3>

                <div class="grid grid-cols-2 gap-x-6 gap-y-4">
                    <div class="flex flex-col gap-2">
                        <div class="flex gap-1">    
                            <label class="font-medium uppercase text-xs text-gray-700">Item Name</label>
                            <span class="text-red-600 relative -top-1" v-if="r$.$fields.shipmentItems.$each[index].$fields.name.$isRequired">*</span>
                        </div>

                        <input v-model="form.shipmentItems[index].name" type="text" class="ring-1 ring-gray-300 rounded outline-none p-2" />
                        <FieldError :errors="r$.$errors.shipmentItems.$each[index].name" />
                        <div><pre class="bg-gray-100 p-1 rounded text-sm">typeof $isRequired: {{ typeof r$.$fields.shipmentItems.$each[index].$fields.name.$isRequired }}</pre></div>
                    </div>
    
                    <div class="flex flex-col gap-2">
                        <div class="flex gap-1">
                            <label class="font-medium uppercase text-xs text-gray-700">Quantity</label>
                            <span class="text-red-600 relative -top-1" v-if="r$.$fields.shipmentItems.$each[index].$fields.quantity.$isRequired">*</span>
                        </div>

                        <input v-model="form.shipmentItems[index].quantity" type="number" class="ring-1 ring-gray-300 rounded outline-none p-2" />
                        <FieldError :errors="r$.$errors.shipmentItems.$each[index].quantity" />
                    </div>
    
                    <div class="flex flex-col gap-2">
                        <div class="flex gap-1">
                            <label class="font-medium uppercase text-xs text-gray-700">Weight</label>
                            <span class="text-red-600 relative -top-1" v-if="r$.$fields.shipmentItems.$each[index].$fields.weight.$isRequired">*</span>
                        </div>
                        
                        <input v-model="form.shipmentItems[index].weight" type="number" class="ring-1 ring-gray-300 rounded outline-none p-2" />
                        <FieldError :errors="r$.$errors.shipmentItems.$each[index].weight" />
                    </div>
                </div>
            </div>

            <button @click="addShipmentItem" class="ml-2 bg-blue-500 text-white px-10 py-3 rounded mt-6 hover:bg-blue-600 transition">
                Add Shipment Item
            </button>

            <button @click="someCondition = !someCondition" class="ml-2 bg-gray-500 text-white px-10 py-3 rounded mt-6 hover:bg-gray-600 transition">
                Toggle Condition: {{ someCondition ? 'ON' : 'OFF' }}
            </button>

            <button @click="someNumber++" class="ml-2 bg-gray-500 text-white px-10 py-3 rounded mt-6 hover:bg-gray-600 transition">
                Some Number: {{ someNumber }}
            </button>
        </div>

        <br />
        <div style="font-size: 13px; max-height:500px;overflow:auto;padding: 10px;background: #f8f8f8;border-radius: 4px;font-family: monospace;"><pre><b>r$:</b> {{ r$ }}</pre></div>
    </div>
</template>

<script setup lang="ts">
    import { defineRegleConfig } from '@regle/core'
    import type { RegleExternalErrorTree } from '@regle/core'
    import { required, minLength, minValue, applyIf, withMessage } from '@regle/rules'

    const { t } = useI18n()

    const someCondition = ref(true)
    const someNumber = ref(4)

    interface Form {
        referenceNumber: string
        shipmentItems: {
            name: string
            quantity: number
            weight: number | string
        }[]
    }

    const form = ref<Form>({
        referenceNumber: '',
        shipmentItems: [
            { name: '', quantity: 0, weight: 0 },
            { name: '', quantity: 0, weight: 0 }
        ]
    })

    const externalErrors = ref<RegleExternalErrorTree<Form>>({
        referenceNumber: ['Backend says reference number is invalid'],
        shipmentItems: {
            $each: [
                {
                    name: ['Backend says shipmentItem[0].name is invalid'],
                },
            ],
        }
    })

    const { useRegle } = defineRegleConfig({
        shortcuts: {
            fields: {
                $isRequired: (field) => !!field.$rules.required?.$active
            }
        }
    })

    const minWeightRule = () => 
        withMessage(
            value => Number(value) >= someNumber.value && someCondition.value === true,
            t('no_good', { name: someNumber.value })
        )

    const rules = computed(() => {
        return {
            referenceNumber: {
                required: applyIf(someCondition, required),
                minLength: applyIf(someCondition, minLength(5))
            },
            shipmentItems: {
                $each: {
                    name: {
                        minWeight: minWeightRule(),
                        required: applyIf(someCondition, required),
                        minLength: applyIf(someCondition, minLength(3))
                    },
                    quantity: {
                        minValue: minValue(1)
                    },
                    weight: {
                        required
                    }
                }
            }
        }
    })

    const { r$ } = useRegle(form, rules, { lazy: false, externalErrors })

    const addShipmentItem = () => {
        form.value.shipmentItems.push({ name: '', quantity: 1, weight: '' })
    }
</script>
