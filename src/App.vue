<script setup lang="ts">
import { onMounted, ref } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/css";

const cardLimit = 300;
const cardIncrease = 9;
const pageCount = Math.ceil(cardLimit / cardIncrease);
let currentPage = 1;
const isLoader = ref(true);
const items = ref<number[][]>([]);

const getRandomColor = () => {
  const h = Math.floor(Math.random() * 360);
  return `hsl(${h}deg, 90%, 85%)`;
};

const handleScroll = (e: any) => {
  console.log(e);
};

function splitArray(array: any[], part: number) {
  var tmp = [];
  for (var i = 0; i < array.length; i += part) {
    tmp.push(array.slice(i, i + part));
  }
  return tmp;
}

var throttleTimer: any = undefined;
const throttle = (callback: any, time: number) => {
  if (throttleTimer) return;

  throttleTimer = true;

  setTimeout(() => {
    callback();
    throttleTimer = false;
  }, time);
};

const flipCardAction = (e: any) => {
  e.currentTarget.classList.toggle("flip");
};

const addCards = (pageIndex: number) => {
  // currentPage = pageIndex;
  // const startRange = (pageIndex - 1) * cardIncrease;
  // const endRange =
  //   currentPage == pageCount ? cardLimit : pageIndex * cardIncrease;

  // const appended = new Array(endRange - startRange).values();
  // console.log(appended);
  items.value.push([1, 2, 3, 4]);
  items.value.push([1, 2, 3, 4]);
  items.value.push([1, 2, 3, 4]);
};

const handleInfiniteScroll = () => {
  throttle(() => {
    const endOfPage =
      window.innerHeight + window.pageYOffset >= document.body.offsetHeight;

    if (endOfPage) {
      addCards(currentPage + 1);
    }

    if (currentPage === pageCount) {
      removeInfiniteScroll();
    }
  }, 1000);
};

const handleReachEnd = (idx: any) => {
  const current = items.value[idx];
  const appendItems = Array.from(new Array(3).keys()).map(
    (i) => current.length + (i + 1)
  );
  items.value[idx] = [...current, ...appendItems];
};

const removeInfiniteScroll = () => {
  isLoader.value = false;
  window.removeEventListener("scroll", handleInfiniteScroll);
};

onMounted(() => {
  addCards(currentPage);
  window.addEventListener("scroll", handleInfiniteScroll);
});

const getCountPrevItems = (idx: number) => {
  if (idx === 0) {
    return 0;
  }
  const prevItems = [...items.value].splice(0, idx);
  return prevItems.reduce((acc, rowItem) => acc + rowItem.length, 0);
};
</script>

<template>
  <main>
    <section>
      <div id="card-container">
        <swiper
          v-for="(item, rowIdx) in items"
          :key="rowIdx"
          :slidesPerView="3"
          :spaceBetween="20"
          :grabCursor="true"
          class="mySwiper"
          @reach-end="handleReachEnd(rowIdx)"
        >
          <swiper-slide v-for="(_, idx2) in item" :key="idx2">
            <div class="card" @click="flipCardAction">
              <div
                class="inner"
                :style="{ 'background-color': getRandomColor() }"
              >
                <div class="front">
                  {{ getCountPrevItems(rowIdx) + (idx2 + 1) }}
                </div>
                <div class="back">back</div>
              </div>
            </div>
          </swiper-slide>
        </swiper>
      </div>
      <div id="loader" v-if="isLoader">
        <div class="skeleton-card"></div>
        <div class="skeleton-card"></div>
        <div class="skeleton-card"></div>
      </div>
    </section>
  </main>
</template>

<style>
.swiper {
  width: 100%;
  margin-bottom: 20px;
  /* height: 100%; */
}

.swiper-slide {
  text-align: center;
  font-size: 18px;
  background: #fff;

  /* Center slide text vertically */
  display: flex !important;
  justify-content: center;
  align-items: center;
  /* border: 1px solid;
  box-sizing: border-box; */
}

.swiper-slide img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.hide {
  display: none !important;
}
#card-container {
  /* display: flex;
  flex-wrap: wrap; */
}

.card-row {
  /* display: flex; */
  /* flex: 1; */
  /* flex-wrap: nowrap; */
  /* width: 1000px; */
  white-space: nowrap;
  overflow-x: auto;
}

.card {
  height: 35vh;
  width: 100%;
  /* width: calc((100% / 3) - 16px); */
  /* display: inline-block; */
  /* width: 300px; */
  /* margin: 8px; */
  /* border-radius: 3px; */
  transition: all 200ms ease-in-out;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  /* cursor: pointer; */

  perspective: 1000px; /* Remove this if you don't want the 3D effect */
}

/* .card:hover {
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
} */
.inner {
  position: relative;
  width: 100%;
  height: 100%;
  text-align: center;
  transition: transform 0.8s;
  transform-style: preserve-3d;
  flex: 1;
}

/* Position the front and back side */
.front,
.back {
  position: absolute;
  width: 100%;
  height: 100%;
  -webkit-backface-visibility: hidden; /* Safari */
  backface-visibility: hidden;
}
.front {
  display: flex;
  align-items: center;
  justify-content: center;
}
.back {
  display: flex;
  align-items: center;
  justify-content: center;
  transform: rotateY(180deg);
}

.card.flip .inner {
  transform: rotateY(180deg);
}

.card-actions {
  margin: 8px;
  padding: 16px 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

#loader {
  display: flex;
}

.skeleton-card {
  height: 35vh;
  width: calc((100% / 3) - 16px);
  margin: 8px;
  border-radius: 3px;
  transition: all 200ms ease-in-out;
  position: relative;
  background-color: #eaeaea;
}

.skeleton-card::after {
  content: "";
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  transform: translateX(-100%);
  background-image: linear-gradient(
    90deg,
    rgba(255, 255, 255, 0) 0,
    rgba(255, 255, 255, 0.2) 20%,
    rgba(255, 255, 255, 0.5) 60%,
    rgba(255, 255, 255, 0)
  );
  animation: load 1s infinite;
}

@keyframes load {
  100% {
    transform: translateX(100%);
  }
}

@media screen and (prefers-reduced-motion: reduce) {
  .skeleton-card::after {
    animation: none;
  }
}
</style>
