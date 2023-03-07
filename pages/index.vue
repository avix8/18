<template>
  <v-row justify="center" align="center">
    <div class="fixed">
      <v-switch v-model="outlined"></v-switch>
      <v-switch v-model="signatures"></v-switch>
      <v-switch v-model="labels"></v-switch>
    </div>
    <v-col>
      <v-row justify="space-between">
        <v-col cols="6">
          <v-row>
            <v-card-title>АКТ</v-card-title>
            <v-col>
              <v-text-field
                :outlined="outlined"
                dense
                hide-details
                :prefix="labels ? '' : '№'"
                :label="labels ? 'Номер документа' : ''"
                :items="[]"
              />
            </v-col>
            <v-col>
              <v-menu
                v-model="dateMenu"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template #activator="{ on, attrs }">
                  <v-text-field
                    v-model="date"
                    :outlined="outlined"
                    label="Дата"
                    hide-details
                    dense
                    append-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="date"
                  locale="ru"
                  @input="dateMenu = false"
                ></v-date-picker>
              </v-menu>
            </v-col>
          </v-row>

          <v-row>
            <v-card-title
              style="word-break: break-word !important"
              class="pt-0 pb-4"
              >О ПЕРЕДАЧЕ ТОВАРОВ И ТАРЫ ПРИ СМЕНЕ МАТЕРИАЛЬНО ОТВЕТСТВЕННОГО
              ЛИЦА</v-card-title
            >
          </v-row>
          <v-row
            class="mx-0 mb-5"
            :style="{ paddingBottom: outlined ? 0 : '10px' }"
          >
            <A-Autocomplete
              v-model="organization"
              :outlined="outlined"
              hide-details
              dense
              label="Организация"
              :items="organizations.map((organization) => organization.name)"
            />
          </v-row>
          <v-row
            class="mx-0 mb-5"
            :style="{ paddingBottom: outlined ? 0 : '10px' }"
          >
            <A-Autocomplete
              v-model="subdivision"
              :outlined="outlined"
              hide-details
              dense
              label="Подразделение"
              :items="
                organization
                  ? organizations.filter((o) => o.name === organization)?.[0]
                      ?.subdivisions
                  : organizations.reduce((acc, cur) => {
                      acc.push(...cur.subdivisions)
                      return acc
                    }, [])
              "
            />
          </v-row>
        </v-col>

        <v-col cols="3"> </v-col>

        <v-col cols="3">
          <v-row
            v-for="detail in documentDetails"
            :key="detail.label"
            class="ma-0 mb-5"
            :style="{ paddingBottom: outlined ? 0 : '10px' }"
          >
            <A-Autocomplete
              v-model="detail.value"
              hide-details
              :outlined="outlined"
              :disabled="detail.disabled"
              dense
              :label="detail.label"
            />
          </v-row>
        </v-col>
      </v-row>

      <v-data-table
        :items="products"
        :headers="headers"
        :search="search"
        :footer-props="{
          showCurrentPage: true,
        }"
        group-by="type"
        group-desc
        dense
      >
        <template #header>
          <thead class="v-data-table-header">
            <tr>
              <th></th>
              <th colspan="2" class="text-center">Товар, тара</th>
              <th colspan="2" class="text-center">Единица измерения</th>
              <th colspan="5" class="px-0">
                <v-text-field
                  v-model="search"
                  append-icon="mdi-magnify"
                  placeholder="Поиск"
                  dense
                  hide-details
                />
              </th>
            </tr>
          </thead>
        </template>

        <template #[`group.header`]="{ group }">
          <td></td>
          <td colspan="2" class="font-weight-bold">{{ group }}</td>
          <td v-for="h in headers.slice(3)" :key="h.name"></td>
        </template>

        <template #[`group.summary`]="groupProps">
          <td></td>
          <td class="text-right">Итого</td>
          <td
            v-for="h in headers.slice(2)"
            :key="h.value"
            :class="{
              [`text-${h.align}`]: true,
              'font-weight-bold': !h.summable,
            }"
          >
            {{
              h.summable
                ? products
                    .filter((p) => p.type === groupProps.group)
                    .reduce((acc, cur) => acc + (Number(cur[h.value]) || 0), 0)
                : 'X'
            }}
          </td>
        </template>

        <template #[`body.append`]>
          <tr class="v-row-group__summary">
            <td></td>
            <td class="text-right">Всего по акту</td>
            <td
              v-for="h in headers.slice(2)"
              :key="h.value"
              :class="{
                [`text-${h.align}`]: true,
                'font-weight-bold': !h.summable,
              }"
            >
              {{
                h.summable
                  ? products.reduce(
                      (acc, cur) => acc + (Number(cur[h.value]) || 0),
                      0
                    )
                  : 'X'
              }}
            </td>
          </tr>
        </template>

        <template #item="{ item }">
          <tr>
            <td
              v-for="h in headers"
              :key="h.value"
              :class="{ [`text-${h.align}`]: true, 'px-0': h.value !== 'n' }"
            >
              <template v-if="h.value === 'n'">{{
                Object.keys(item).length > 2 ? item.n : ''
              }}</template>

              <v-icon
                v-else-if="h.value === 'actions'"
                small
                @click="deleteItem(item)"
              >
                {{ Object.keys(item).length > 2 ? 'mdi-delete' : '' }}
              </v-icon>

              <A-Autocomplete
                v-else
                v-model="getItem(item)[h.value]"
                hide-details
                dense
                :clearable="false"
                :align="h.align"
                :items="getSuggestions(h.value, item)"
              />
            </td>
          </tr>
        </template>
      </v-data-table>
      <v-col></v-col>
      <template v-if="signatures">
        <v-row v-for="personData in personsData" :key="personData.label">
          <v-col class="py-0">
            <v-subheader>{{ personData.label }}</v-subheader>
          </v-col>

          <v-col :class="{ 'py-0': true, 'pb-3': !outlined }"
            ><A-Autocomplete
              v-model="personData.position"
              :outlined="outlined"
              dense
              hide-details
              label="Должность"
              :items="positions.map((p) => p.title)"
          /></v-col>
          <v-col :class="{ 'py-0': true, 'pb-3': !outlined }">
            <A-Autocomplete
              v-model="personData.fullname"
              :outlined="outlined"
              dense
              hide-details
              label="ФИО"
              :items="persons.map((p) => p.fullname)"
            />
          </v-col>
        </v-row>
      </template>

      <v-dialog v-else v-model="signaturesDialog" width="1177">
        <template #activator="{ on, attrs }">
          <v-btn color="primary" v-bind="attrs" v-on="on"> Подписи </v-btn>
        </template>
        <v-card>
          <v-card-title>Подписи</v-card-title>
          <v-card-text>
            <v-row
              v-for="personData in personsData"
              :key="personData.label"
              class="my-0"
            >
              <v-col class="py-0">
                <v-subheader>{{ personData.label }}</v-subheader>
              </v-col>

              <v-col :class="{ 'py-0': true, 'pb-3': !outlined }"
                ><A-Autocomplete
                  v-model="personData.position"
                  :outlined="outlined"
                  dense
                  hide-details
                  label="Должность"
                  :items="positions.map((p) => p.title)"
              /></v-col>
              <v-col :class="{ 'py-0': true, 'pb-3': !outlined }">
                <A-Autocomplete
                  v-model="personData.fullname"
                  :outlined="outlined"
                  dense
                  hide-details
                  label="ФИО"
                  :items="persons.map((p) => p.fullname)"
                />
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-dialog>

      <v-row class="my-5 mx-0">
        <v-spacer />
        <v-btn> Экспорт&nbsp;<v-icon>mdi-export</v-icon> </v-btn>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'IndexPage',
  data: () => ({
    outlined: true,
    signatures: true,
    labels: true,
    search: '',
    date: '',
    organization: '',
    subdivision: '',
    dateMenu: false,
    signaturesDialog: false,
    documentDetails: [
      { label: 'ОКУД', value: '0330518', disabled: true },
      { label: 'ОКПО', value: '' },
      { label: 'Вид деятельности по ОКДП', value: '' },
      { label: 'Вид операции', value: '' },
    ],
    personsData: [
      { label: 'Сдал', position: '', fullname: '' },
      { label: 'Принял', position: '', fullname: '' },
      { label: 'Представитель администрации', position: '', fullname: '' },
    ],
    headers: [
      { text: '№', value: 'n', align: 'end', sortable: false },
      { text: 'наименование', value: 'name' },
      { text: 'код', value: 'SKU', align: 'end' },
      { text: 'наименование', value: 'unitName', align: 'center' },
      { text: 'код по ОКЕЙ', value: 'OKEI', align: 'right' },
      { text: 'Масса', value: 'weight', align: 'end', summable: true },
      { text: 'Количество', value: 'amount', align: 'end', summable: true },
      { text: 'Цена, руб. коп.', value: 'price', align: 'end' },
      {
        text: 'Сумма, руб. коп.',
        value: 'total',
        align: 'end',
        summable: true,
      },
      { text: 'Действия', value: 'actions', align: 'center', sortable: false },
    ],
    organizations: [
      { name: 'ООО "СОЛНЫШКО"', subdivisions: ['Столовая №1', 'Столовая №2'] },
      {
        name: 'ООО "Хлеб да соль"',
        subdivisions: ['Столовая АлтГТУ', 'Столовая какая-то'],
      },
    ],
    products: [
      {
        type: 'Товары',
        name: 'Цыплята-бройлеры',
        SKU: '1',
        unitName: 'кг',
        OKEI: '166',
        weight: '20',
        amount: '17',
        price: '80',
        total: '1600',
      },
      {
        type: 'Товары',
        name: 'Окорочка',
        SKU: '2',
        unitName: 'кг',
        OKEI: '166',
        weight: '5',
        amount: '7',
        price: '100',
        total: '500',
      },
      {
        type: 'Товары',
        name: 'Голень цыплёнка',
        SKU: '3',
        unitName: 'кг',
        OKEI: '166',
        weight: '4',
        amount: '4',
        price: '120',
        total: '480',
      },
      {
        type: 'Товары',
        name: 'Грудки цыплёнка',
        SKU: '4',
        unitName: 'кг',
        OKEI: '166',
        weight: '5',
        amount: '5',
        price: '120',
        total: '600',
      },
      {
        type: 'Товары',
        name: 'Бедро цыплёнка',
        SKU: '5',
        unitName: 'кг',
        OKEI: '166',
        weight: '3',
        amount: '3',
        price: '120',
        total: '360',
      },
      {
        type: 'Тара',
        name: 'Контейнер пластиковый',
        SKU: '403',
        unitName: 'шт',
        OKEI: '796',
        weight: '1',
        amount: '5',
        price: '50',
        total: '250',
      },
    ],

    productNames: [''],
    positions: [
      { title: 'Кладовщик', code: 1 },
      { title: 'Заведующий производством', code: 2 },
      { title: 'Управляющий', code: 3 },
    ],
    persons: [
      { fullname: 'Замков В.И.', position: 1 },
      { fullname: 'Деловой П.В.', position: 2 },
      { fullname: 'Веселова М.Н.', position: 3 },
    ],
    units: {
      шт: ['796'],
      кг: ['166'],
    },
    types: ['Товары', 'Тара'],
  }),
  watch: {
    products() {
      this.addEmptyIfNotExist()
    },
  },
  mounted() {
    this.recalculateNumbers()
    this.addEmptyIfNotExist()
  },
  methods: {
    addEmptyIfNotExist() {
      const empty = this.products.reduce(
        (acc, p) => {
          acc[p.type] += Object.keys(p).length < 3
          return acc
        },
        this.types.reduce((acc, t) => {
          acc[t] = 0
          return acc
        }, {})
      )
      this.types.forEach((type) => {
        if (empty[type] < 1) {
          this.addItem(type)
        }
      })
    },
    recalculateNumbers() {
      const n = {}
      this.products.forEach((p) => {
        n[p.type] = (n[p.type] || 0) + 1
        if (p.n !== n[p.type]) p.n = n[p.type]
      })
    },
    getItem(itemProps) {
      return this.products.find(
        (p) => p.n === itemProps.n && p.type === itemProps.type
      )
    },
    addItem(type) {
      this.products.push({ type })
      this.recalculateNumbers()
    },
    deleteItem(item) {
      this.products = this.products.filter(
        (p) => p.n !== item.n || p.type !== item.type
      )
      this.recalculateNumbers()
    },
    getSuggestions(value, item) {
      if (value === 'total')
        return [
          item.amount * item.price || '',
          item.weight * item.price || '',
        ].map(String)
      if (value === 'unitName') return Object.keys(this.units)
      if (value === 'OKEI') return this.units[item.unitName] || []
      return []
    },
  },
}
</script>

<style>
td,
th {
  border: 1px solid lightgray !important;
}

.v-input__slot {
  padding: 0 16px;
}

.fixed {
  position: fixed;
  top: 0;
  left: 0;
}
</style>
