<script setup lang="ts">
import { ref } from 'vue'
import "@fontsource/roboto"; // Defaults to weight 400
import 'primeicons/primeicons.css';
import Card from 'primevue/card';
import InputText from 'primevue/inputtext';
import IftaLabel from 'primevue/iftalabel';
import Button from 'primevue/button';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Fieldset from 'primevue/fieldset';
import MultiSelect from 'primevue/multiselect';

interface Item {
  name: string
  owners: Person[]
  price: string
}

interface SplitItem {
  name: string
  partialPrice: number
}

interface Person {
  name: string
}

interface PersonTotal {
  name: string
  items: SplitItem[]
  subtotal: number
  tax: number
  tip: number
  total: number
}

// Bill Totals
const tax = ref<string>('6.625');
const tip = ref<string>('20');
const total = ref<string>('');

// Person
const person = ref<string>('');
const people = ref<Person[]>([]);

// Item
const item = ref<string>('');
const price = ref<string>('0');
const quantity = ref<string>('1');
const items = ref<Item[]>([]);
const owners = ref<Person[]>([]);

// Summary
const results = ref<PersonTotal[]>([]);

const resetForm = () => {
  item.value = "";
  quantity.value = '1';
  price.value = '0';
  owners.value = [];
};

const addPerson = (event: SubmitEvent) => {
  event.preventDefault();
  const personToAdd = person.value.trim();
  if (people.value.find(p => p.name == personToAdd)) {
    return;
  }
  people.value.push({ name: personToAdd });
  person.value = '';
};

const removePerson = (index: number) => {
  people.value.splice(index, 1)
};

const addItem = (event: SubmitEvent) => {
  event.preventDefault();
  let itemPrice = Number(price.value);
  let itemQuantity = Number(quantity.value);

  if (itemQuantity < 1) {
    return
  } else if (itemQuantity > 1) {
    itemPrice = itemPrice / itemQuantity;
  }

  for (let i = 0; i < itemQuantity; i++) {
    items.value.push({ name: item.value, owners: owners.value, price: itemPrice.toFixed(2) });
  }

  resetForm();
};

const removeItem = (index: number) => {
  items.value.splice(index, 1)
};

const splitBill = () => {
  results.value.length = 0
  const totalsByPerson = new Map<string, PersonTotal>()
  totalsByPerson.clear()
  for (const person of people.value) {
    totalsByPerson.set(person.name, {
      items: [],
      name: person.name,
      subtotal: 0,
      tax: 0,
      tip: 0,
      total: 0,
    })
  }

  for (const item of items.value) {
    const splitPrice = Number(item.price) / item.owners.length

    for (const owner of item.owners) {
      totalsByPerson.get(owner.name)?.items.push({
        name: owner.name,
        partialPrice: Number(splitPrice.toFixed(2))
      })
    }
  }

  const taxAsDecimal = Number(tax.value) / 100
  const tipAsDecimal = Number(tip.value) / 100

  for (const [person, total] of totalsByPerson) {
    // Add all the items for the person
    total.subtotal = total.items.map(i => i.partialPrice).reduce((a, b) => a + b, 0)
    total.tax = total.subtotal * taxAsDecimal
    total.tip = total.subtotal * tipAsDecimal
    total.total = total.subtotal + total.tax + total.tip

    results.value.push(total)

    console.log(`${person} 
    sub: ${total.subtotal.toFixed(2)}
    tax: ${total.tax.toFixed(2)}
    tip: ${total.tip.toFixed(2)}
    tot: ${total.total.toFixed(2)}
    `)
  }
};

</script>

<template>
  <Card>
    <template #title>
      Split Bill
    </template>
    <template #content>

      <Fieldset legend="Define People">
        <form @submit="addPerson">
          <div class="form-control">
            <IftaLabel>
              <InputText required="true" id="item" v-model="person" />
              <label for="item">Person (name)</label>
            </IftaLabel>
          </div>
          <Button type="submit" label="Save" icon="pi pi-check" iconPos="right" />
        </form>
        <DataTable v-if="people.length" :value="people" tableStyle="min-width: 50rem">
          <Column field="name" header="Name">
            <template #body="slotProps">
              <InputText type="text" v-model="slotProps.data.name" />
            </template>
          </Column>
          <Column>
            <template #body="slotProps">
              <Button type="button" @click="removePerson(slotProps.index)" aria-label="Remove Person" icon="pi pi-trash"
                severity="danger" />
            </template>
          </Column>
        </DataTable>
      </Fieldset>

      <Fieldset legend="Define Totals">
        <div class="form-control">
          <IftaLabel>
            <InputText required="true" id="tax" type="number" v-model="tax" />
            <label for="tax">Tax (%)</label>
          </IftaLabel>
        </div>
        <div class="form-control">
          <IftaLabel>
            <InputText required="true" id="tip" type="number" v-model="tip" />
            <label for="tip">Tip (%)</label>
          </IftaLabel>
        </div>
        <div class="form-control">
          <IftaLabel>
            <InputText required="true" id="total" type="number" v-model="total" />
            <label for="total">Total ($)</label>
          </IftaLabel>
        </div>
      </Fieldset>

      <Fieldset legend="Add Items">
        <form @submit="addItem">
          <div class="form-control">
            <IftaLabel>
              <InputText required="true" id="item" v-model="item" />
              <label for="item">Item Name</label>
            </IftaLabel>
          </div>
          <div class="form-control">
            <IftaLabel>
              <InputText id="price" type="number" v-model="price" />
              <label for="price">Price</label>
            </IftaLabel>
          </div>
          <div class="form-control">
            <IftaLabel>
              <InputText id="quantity" type="number" v-model="quantity" />
              <label for="quantity">Quantity</label>
            </IftaLabel>
          </div>
          <div class="form-control">
            <IftaLabel>
              <MultiSelect id="owners" v-model="owners" :options="people" optionLabel="name" filter
                placeholder="Select Owners" :maxSelectedLabels="3" class="w-full md:w-80" />
              <label for="owners">Owners</label>
            </IftaLabel>
          </div>
          <Button type="submit" label="Save" icon="pi pi-check" iconPos="right" />
        </form>

        <DataTable v-if="items.length" :value="items" tableStyle="min-width: 50rem">
          <Column field="name" header="Item">
            <template #body="slotProps">
              <InputText type="text" v-model="slotProps.data.name" />
            </template>
          </Column>
          <Column field="price" header="Price">
            <template #body="slotProps">
              <InputText type="number" v-model="slotProps.data.price" />
            </template>
          </Column>
          <Column field="owners" header="Owners">
            <template #body="slotProps">
              <MultiSelect id="owners" v-model="slotProps.data.owners" :options="people" optionLabel="name" filter
                placeholder="Select Owners" :maxSelectedLabels="3" class="w-full md:w-80" />
            </template>
          </Column>
          <Column>
            <template #body="slotProps">
              <Button type="button" @click="removeItem(slotProps.index)" aria-label="Remove Item" icon="pi pi-trash"
                severity="danger" />
            </template>
          </Column>
        </DataTable>
      </Fieldset>

      <Button type="button" @click="splitBill" label="SPLIT!" icon="pi pi-check" iconPos="right" />

      <Fieldset v-if="results.length" header="Results">
        <DataTable :value="results" tableStyle="min-width: 50rem">
          <Column field="name" header="Name"></Column>
          <Column field="subtotal" header="Subtotal"></Column>
          <Column field="tax" header="Tax"></Column>
          <Column field="tip" header="Tip"></Column>
          <Column field="total" header="Total"></Column>
        </DataTable>
      </Fieldset>
    </template>
  </Card>
</template>

<style scoped>
.form-control {
  margin-bottom: 1em;
}
</style>
