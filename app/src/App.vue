<script setup lang="ts">
  // standard libs
  import { ref, reactive, computed } from 'vue';
  // internal libs
  import dishesData from '../src/data/dished.json';
  // external libs
  import { Minus, Plus } from '@element-plus/icons-vue';
  import { toast } from 'vue3-toastify';
  import 'vue3-toastify/dist/index.css';

  // state
  const active = ref(1);
  const steps = [
    {
      title: 'Step1',
    },
    {
      title: 'Step2',
    },
    {
      title: 'Step3',
    },
    {
      title: 'Review',
    },
  ];
  const dishes = dishesData.dishes;
  const order = reactive({
    mealValue: null,
    numberOfPeople: 0,
    restaurant: null,
    dishes: [],
  });
  const selectedDish = ref('');
  const numberOfServing = ref(1);

  // computed
  const availableMeals = computed(() => {
    const mealsSet = new Set();

    // Lặp qua mảng dishes để lấy tất cả các bữa ăn có sẵn
    dishes.forEach((dish) => {
      dish.availableMeals.forEach((meal) => {
        mealsSet.add(meal);
      });
    });

    // Chuyển đổi Set thành mảng và trả về
    return Array.from(mealsSet);
  });
  const availableRestaurants = computed(() => {
    const selectedMeal = order.mealValue; // Giả sử order.mealValue là giá trị từ v-model

    if (!selectedMeal) return;
    const filteredRestaurants = dishes.filter((dish) =>
      dish.availableMeals.includes(selectedMeal),
    );

    const uniqueRestaurants = Array.from(
      new Set(filteredRestaurants.map((dish) => dish.restaurant)),
    ).map((restaurantName) =>
      filteredRestaurants.find((dish) => dish.restaurant === restaurantName),
    );

    return uniqueRestaurants;
  });
  const selectedRestaurant = computed(() => {
    const selectedMeal = order.mealValue;
    const selectedRestaurantName = order.restaurant;

    if (selectedMeal === null || selectedRestaurantName === null) {
      return [];
    }

    // Lọc danh sách các món ăn của nhà hàng đã chọn có bữa ăn cụ thể
    return dishes.filter(
      (dish) =>
        dish.restaurant === selectedRestaurantName &&
        dish.availableMeals.includes(selectedMeal),
    );
  });
  const isStepValid = computed(() => {
    if (active.value === 1) {
      return order.mealValue !== null && order.numberOfPeople !== 0;
    } else if (active.value === 2) {
      return order.restaurant !== null;
    } else if (active.value === 3) {
      return order.dishes.length > 0;
    }

    // Default to true if step is not recognized
    return true;
  });

  /**
   * TODO: previous button click handler
   */
  const prev = () => active.value > 1 && active.value--;
  /**
   * TODO: change the active tab handler click
   */
  const next = () => active.value < steps.length && active.value++;

  const onDecreasePeoplep = () => {
    order.numberOfPeople--;
  };
  const onIncreasePeople = () => {
    order.numberOfPeople++;
  };
  /**
   * TODO: Next button click handler
   */
  const proceedToNextStep = () => {
    if (isStepValid.value) {
      next();
    } else {
      toast('Validation error: Please fill in the required fields.', {
        autoClose: 2000,

        theme: 'colored',
        type: 'warning',
        pauseOnHover: false,
        transition: 'slide',
        dangerouslyHTMLString: true,
      });
    }
  };

  /**
   * TODO: add dishs and number of servings to order
   */
  const onAddDish = () => {
    // Assuming selectedDish and numberOfServing are refs
    if (!selectedDish.value || !numberOfServing.value) return;

    // Assuming dish is a string and number is a number
    let dish: string = selectedDish.value!;
    let number: number = numberOfServing.value!;
    let existingDish = order.dishes.find((item) => item.dish === dish);
    if (existingDish) {
      toast('This dish has been selected, please choose another dish', {
        autoClose: 2000,

        theme: 'colored',
        type: 'warning',
        pauseOnHover: false,
        transition: 'slide',
        dangerouslyHTMLString: true,
      });
      return;
    }
    // Assuming order.dishes is an array of objects with 'dish' and 'number' properties
    order.dishes.push({
      dish,
      number,
    });

    selectedDish.value = '';
    numberOfServing.value = 1;
  };

  /**
   * TODO: submit order
   */
  const onSubmit = () => {
    toast('Your order has bees sent, please wait...', {
      type: 'success',
      autoClose: 2000,

      theme: 'colored',
      pauseOnHover: false,
      transition: 'slide',
      dangerouslyHTMLString: true,
    });

    order.mealValue = null;
    order.numberOfPeople = 1;
    order.restaurant = null;
    order.dishes = [];

    active.value = 1;
  };
</script>

<template>
  <el-steps
    class="my-8 mx-auto"
    style="max-width: 600px"
    :space="200"
    :active="active"
    align-center
  >
    <el-step
      v-for="step in steps"
      :key="step.title"
      :title="step.title"
    />
  </el-steps>
  <div class="content p-[100px]">
    <div
      class="step1"
      v-if="active === 1"
    >
      <h2>Please select meal <span class="text-[#fd3838]">*</span></h2>
      <el-select
        class="mb-4"
        v-model="order.mealValue"
        clearable
        placeholder="Select meal"
        style="width: 240px"
      >
        <el-option
          v-for="meal in availableMeals"
          :key="meal"
          :label="meal"
          :value="meal"
        />
      </el-select>
      <h2>
        Please select number of people <span class="text-[#fd3838]">*</span>
      </h2>
      <div class="input--group mb-4 flex gap-2">
        <el-button
          class="mr-4"
          :icon="Minus"
          style="background-color: cornflowerblue; color: white"
          @click="onDecreasePeoplep"
        />

        <el-input
          v-model="order.numberOfPeople"
          max="10"
          min="1"
          type="number"
          style="width: 240px"
          placeholder="Please input number of people"
        />
        <el-button
          class="ml-4"
          :icon="Plus"
          style="background-color: cornflowerblue; color: white"
          @click="onIncreasePeople"
        />
      </div>
    </div>
    <div
      class="step2"
      v-if="active === 2"
    >
      <h2>Please select restaurent <span class="text-[#fd3838]">*</span></h2>
      <el-select
        v-model="order.restaurant"
        class="mb-4"
        clearable
        placeholder="Select restaurant"
        style="width: 240px"
      >
        <el-option
          v-for="restaurant in availableRestaurants || []"
          :key="restaurant?.restaurant"
          :label="restaurant?.restaurant || 'Unknown Restaurant'"
          :value="restaurant?.restaurant || 'Unknown Restaurant'"
        />
      </el-select>
    </div>
    <div
      class="step3"
      v-if="active === 3"
    >
      <div class="content flex gap-10">
        <div class="left">
          <h2>Please select dish <span class="text-[#fd3838]">*</span></h2>
          <el-select
            v-model="selectedDish"
            clearable
            placeholder="Select dishes"
            style="width: 240px"
          >
            <el-option
              v-for="restaurant in selectedRestaurant || []"
              :key="restaurant?.name"
              :label="restaurant?.name || 'Unknown Restaurant'"
              :value="restaurant?.name || 'Unknown Restaurant'"
            />
          </el-select>
          <el-button
            :icon="Plus"
            circle
            class="mx-4"
            @click="onAddDish"
          />
        </div>
        <div class="right">
          <h2>
            Please enter no. of serving <span class="text-[#fd3838]">*</span>
          </h2>
          <el-input
            type="number"
            v-model="numberOfServing"
          >
          </el-input>
        </div>
        <div class="total">
          <h3>Dish and serves</h3>
          <li
            v-for="item in order.dishes"
            :key="item.dish"
          >
            {{ item.dish }} - {{ item.number }}
          </li>
        </div>
      </div>
    </div>
    <div
      class="step4"
      v-if="active === 4"
    >
      <div class="itemOrder mb-4 flex gap-4">
        <h4 class="text-left w-[150px] font-semibold">Meal</h4>
        <div class="context">{{ order.mealValue }}</div>
      </div>
      <div class="itemOrder mb-4 flex gap-4">
        <h4 class="text-left w-[150px] font-semibold">No. of people</h4>
        <div class="context">{{ order.numberOfPeople }}</div>
      </div>
      <div class="itemOrder mb-4 flex gap-4">
        <h4 class="text-left w-[150px] font-semibold">Restaurant</h4>
        <div class="context">{{ order.restaurant }}</div>
      </div>
      <div class="itemOrder mb-4 flex gap-4">
        <h4 class="text-left w-[150px] font-semibold">Dishes</h4>
        <div class="context border-2 border-black p-2">
          <ul>
            <li v-for="dish in order.dishes">
              {{ dish.dish }} - {{ dish.number }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
  <div class="step-action px-20 py-10 w-8/12 flex justify-between">
    <el-button
      style="margin-top: 12px"
      @click="prev"
      v-show="active !== 1"
      >Prev step</el-button
    >
    <Transition name="fade">
      <el-button
        v-if="active < 4"
        style="margin-top: 12px"
        @click="proceedToNextStep"
        >Next step</el-button
      >
      <el-button
        v-else
        style="margin-top: 12px"
        @click="onSubmit"
        >Submit</el-button
      >
    </Transition>
  </div>
</template>

<style scoped>
  .v-enter-active,
  .v-leave-active {
    transition: opacity 0.5s ease;
  }

  .v-enter-from,
  .v-leave-to {
    opacity: 0;
  }
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.5s ease-in;
  }

  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }
</style>
