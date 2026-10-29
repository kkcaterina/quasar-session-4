<script setup>
import { ref, computed, nextTick } from "vue";

const PASS_GRADE = 60;

const students = ref([
  { id: 1, name: "Ali", grade: 80 },
  { id: 2, name: "Sara", grade: 90 },
  { id: 3, name: "Omar", grade: 55 },
]);
let nextId = 4;

const formRef = ref(null);
const newName = ref("");
const newGrade = ref(null);
const search = ref("");

const nameRules = [
  (val) => (val && val.trim().length >= 2) || "Имя: минимум 2 символа",
];
const gradeRules = [
  (val) =>
    (val !== null && val !== "" && val >= 0 && val <= 100) ||
    "Оценка: число от 0 до 100",
];

const filteredStudents = computed(() => {
  const query = (search.value || "").trim().toLowerCase();
  return students.value.filter((s) => s.name.toLowerCase().includes(query));
});

const total = computed(() => students.value.length);

const passedCount = computed(
  () => students.value.filter((s) => s.grade >= PASS_GRADE).length,
);

const average = computed(() => {
  if (students.value.length === 0) return 0;
  const sum = students.value.reduce((acc, s) => acc + s.grade, 0);
  return sum / students.value.length;
});

function onReset() {
  newName.value = "";
  newGrade.value = null;
}

function addStudent() {
  students.value.push({
    id: nextId++,
    name: newName.value.trim(),
    grade: newGrade.value,
  });
  onReset();
  nextTick(() => formRef.value.resetValidation());
}

function removeStudent(id) {
  students.value = students.value.filter((s) => s.id !== id);
}

function changeGrade(student, delta) {
  student.grade = Math.min(100, Math.max(0, student.grade + delta));
}
</script>

<template>
  <q-page padding>
    <div class="page">
      <h1 class="text-h5 q-mt-none q-mb-md">Студенты</h1>
      <q-card flat bordered class="q-mb-md">
        <q-card-section>
          <q-form
            ref="formRef"
            class="row q-col-gutter-md"
            @submit="addStudent"
            @reset="onReset"
          >
            <div class="col-12 col-sm-5">
              <q-input
                v-model="newName"
                label="Имя"
                outlined
                dense
                lazy-rules
                :rules="nameRules"
              />
            </div>
            <div class="col-12 col-sm-3">
              <q-input
                v-model.number="newGrade"
                type="number"
                label="Оценка"
                outlined
                dense
                lazy-rules
                :rules="gradeRules"
              />
            </div>
            <div class="col-12 col-sm-4 q-gutter-sm">
              <q-btn type="submit" color="primary" label="Добавить" no-caps />
              <q-btn type="reset" flat label="Очистить" no-caps />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
      <div class="row q-col-gutter-md q-mb-md">
        <div class="col-4">
          <q-card flat bordered>
            <q-card-section class="text-center">
              <div class="text-h5">{{ total }}</div>
              <div class="text-caption text-grey-7">Всего</div>
            </q-card-section>
          </q-card>
        </div>
        <div class="col-4">
          <q-card flat bordered>
            <q-card-section class="text-center">
              <div class="text-h5">{{ passedCount }}</div>
              <div class="text-caption text-grey-7">Сдали</div>
            </q-card-section>
          </q-card>
        </div>
        <div class="col-4">
          <q-card flat bordered>
            <q-card-section class="text-center">
              <div class="text-h5">{{ average.toFixed(1) }}</div>
              <div class="text-caption text-grey-7">Средний балл</div>
            </q-card-section>
          </q-card>
        </div>
      </div>
      <q-input
        v-model="search"
        label="Поиск по имени"
        outlined
        dense
        clearable
        class="q-mb-md"
      >
        <template v-slot:prepend>
          <q-icon name="search" />
        </template>
      </q-input>
      <q-list v-if="filteredStudents.length" bordered separator>
        <q-item v-for="s in filteredStudents" :key="s.id">
          <q-item-section>
            <q-item-label>{{ s.name }}</q-item-label>
          </q-item-section>
          <q-item-section side>
            <q-badge
              :color="s.grade >= PASS_GRADE ? 'positive' : 'negative'"
              :label="s.grade"
            />
          </q-item-section>
          <q-item-section side>
            <div class="row no-wrap items-center">
              <q-btn
                flat
                dense
                round
                icon="remove"
                aria-label="Минус 5"
                @click="changeGrade(s, -5)"
              />
              <q-btn
                flat
                dense
                round
                icon="add"
                aria-label="Плюс 5"
                @click="changeGrade(s, 5)"
              />
              <q-btn
                flat
                dense
                round
                color="negative"
                icon="delete"
                aria-label="Удалить"
                @click="removeStudent(s.id)"
              />
            </div>
          </q-item-section>
        </q-item>
      </q-list>
      <q-banner v-else rounded class="bg-grey-2 text-grey-8">
        {{ students.length === 0 ? "Список пуст" : "Ничего не найдено" }}
      </q-banner>
    </div>
  </q-page>
</template>

<style scoped>
.page {
  max-width: 720px;
  margin: 0 auto;
}
</style>
