<template>
  <div>
    <!--modal-->
    <div class="modalBackgound" v-if="isModalOpen">
      <div class="modalPage">
        <span class="modalClose" @click="closeModal">&times;</span>
        <h2>{{  modalContent.title }}</h2>
        <p>
          {{  modalContent.summary }}
        </p>
      </div>
    </div>
    <!--modal end-->

    <!-- <div class="dateSetDiv">
      <span class="dateSetText">기간 설정</span>
      <div class="calenderDiv">
        <input type="date" v-model="startDate" />
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="25"
          viewBox="0 0 24 25"
          fill="none"
        >
          <path
            d="M15.586 11.1569L11.636 7.20692C11.4538 7.01832 11.353 6.76571 11.3553 6.50352C11.3576 6.24132 11.4628 5.99051 11.6482 5.8051C11.8336 5.61969 12.0844 5.51452 12.3466 5.51224C12.6088 5.50997 12.8614 5.61076 13.05 5.79292L18.707 11.4499C18.8002 11.5426 18.8741 11.6527 18.9246 11.7741C18.9751 11.8954 19.001 12.0255 19.001 12.1569C19.001 12.2883 18.9751 12.4184 18.9246 12.5398C18.8741 12.6611 18.8002 12.7713 18.707 12.8639L13.05 18.5209C12.9578 18.6164 12.8474 18.6926 12.7254 18.745C12.6034 18.7974 12.4722 18.825 12.3394 18.8262C12.2066 18.8273 12.0749 18.802 11.952 18.7517C11.8291 18.7015 11.7175 18.6272 11.6236 18.5333C11.5297 18.4394 11.4555 18.3278 11.4052 18.2049C11.3549 18.082 11.3296 17.9503 11.3307 17.8175C11.3319 17.6847 11.3595 17.5535 11.4119 17.4315C11.4643 17.3095 11.5405 17.1992 11.636 17.1069L15.586 13.1569H6C5.73478 13.1569 5.48043 13.0516 5.29289 12.864C5.10536 12.6765 5 12.4221 5 12.1569C5 11.8917 5.10536 11.6373 5.29289 11.4498C5.48043 11.2623 5.73478 11.1569 6 11.1569H15.586Z"
            fill="#697077"
          />
        </svg>
        <input type="date" v-model="endDate" />
      </div>
    </div> -->

    <div class="newsListBox">
      <div class="categoryBox">
        <span
          v-for="category in categories"
          :key="category.id"
          class="categoryTitle"
          :class="{ activeCategory: selectedCategory === category.id }"
          @click="selectCategory(category.id)"
        >
          {{ category.name }}
        </span>
      </div>

      <div class="dropdownBox">
        <div class="dropdown">
          <div class="list">
            <div
              class="list-item"
              v-for="(item, index) in filteredNewsItems"
              :key="index"
            >
              <input
                type="checkbox"
                :id="'item' + index"
                v-model="item.isSelected"
              />
              <label :for="'item' + index">
                <div class="item-title">{{ item.title }}</div>
                <a :href="item.url">{{ item.url }}</a>
              </label>
              <div class="readmore" @click="summarizeSelectedArticle(item)">
                요약 보기
              </div>
            </div>
          </div>
        </div>

        <div class="newscontain">
          <WhiteButton @click="addToCart()" ButtonText="뉴스 담기" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import WhiteButton from "../components/WhiteButton.vue";
import axios from "axios";

export default {
  components: { WhiteButton },
  data() {
    return {
      newsItems: [],
      selectedCategory: "1",
      isModalOpen: false,
      summaryContent: "",
      modalContent: {title:"", summary: "", }
    };
  },
  mounted() {
    // 컴포넌트가 마운트되면 데이터를 가져오도록 설정
    this.fetchData();
  },
  computed: {
    categories() {
      return [
        { id: "1", name: "산업정책" },
        { id: "2", name: "건설정책" },
        { id: "3", name: "조선정책" },
        { id: "4", name: "IT 정책" },
      ];
    },
    /* eslint-disable */
    cartItems() {
      return this.$store.state.cartItems;
    },
    filteredNewsItems() {
      return this.newsItems.filter(
        (item) => item.category_id === parseInt(this.selectedCategory)
      );
    },
  },
  watch: {
    startDate() {
      this.handleDateSet();
    },
    endDate() {
      this.handleDateSet();
    },
  },
  methods: {
    // modal
    openModal(item) {
      this.selectedNews = { ...item };
      this.isModalOpen = true;
    },
    closeModal() {
      this.isModalOpen = false;
    },
    //modal end

    selectCategory(category_id) {
      this.selectedCategory = category_id;
      console.log(category_id);
    },

    addToCart() {
      const selectedItems = this.filteredNewsItems.filter(
        (item) => item.isSelected
      );

      if (selectedItems.length > 0) {
        const newItems = selectedItems.filter((item) => {
          // Check if the item is not already in the cart
          return !this.$store.state.cartItems.some(
            (cartItem) => cartItem.title === item.title
          );
        });

        if (newItems.length === 0) {
          window.alert("선택한 뉴스는 이미 담겨 있습니다.");
          return;
        }

        this.$store.commit("addToCart", newItems);

        const confirmAddToCart = window.confirm(
          "선택한 뉴스가 담겼습니다. 담은 뉴스를 장바구니에서 확인하시겠습니까?"
        );

        if (confirmAddToCart) {
          this.$router.push({ name: "BasketPage" });
        }
      } else {
        window.alert("뉴스를 선택해주세요.");
      }
      console.log(this.$store.state.cartItems);
    },

    //title과 link 들고 오는 axios get
    fetchData() {
      axios
        .get("http://localhost:3000/api/clipped-news/specific", {
          params: { category_id: this.selectedCategory },
        })
        .then((response) => {
          // Update newsItems with the fetched data
          this.newsItems = response.data;
        })
        .catch((error) => {
          console.error("API Error:", error);
        });
    },

    summarizeSelectedArticle(item) {
      // Prepare data for the selected article
      const articleData = {
        title: item.title,
        url: item.url,
      };

      // Send a POST request to the server for summarization
      axios
        .post("http://localhost:1004/api/clipped-news/summary", articleData)
        .then((response) => {
          // Handle the response, for example, update the summary state for the selected article
          // console.log("Title:", title);
          // console.log("URL:", url);
          console.log("success:", response.data);
          if (response.data) {
            this.modalContent = {
              title: item.title,
              summary: response.data
            };

            this.isModalOpen = true;
          }
          this.isModalOpen = true;

          // You can update the state or display the title, url, and summary as needed
        })
        .catch((error) => {
          // Handle errors
          console.error("Error during API call", error);
        });
    },
  },
};
</script>

<style scoped>
/* modal */
.modalBackgound {
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  top: 0;
  left: 0;
  position: fixed;
  z-index: 1;
}
.modalPage {
  font-size: 1.25em;
  background: white;
  border-radius: 8px;
  padding: 3%;
  margin: 10%;
  white-space: normal;
}
.modalClose {
  font-size: 2em;
  cursor: pointer;
  position: relative;
  float: right;
}
/* modal end */

input[type="date"] {
  width: 50%;
  height: 5vh;
  text-align: center;
  font-size: 1.5em;
}
.readmore {
  margin-left: auto;
  text-align: right;
  padding: 1%;
  background-color: #aaa;
  color: #fff;
}
.dateSetDiv {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 3%;
  border-bottom: 1px solid #d6dbe5;
}
.dateSetText {
  color: #3b3b3b;
  font-size: 1.5em;
  font-weight: 700;
  margin-bottom: 20px;
}
.calenderDiv {
  display: flex;
  align-items: center;
}
input[type="checkbox"] {
  width: 20px;
  height: 20px;
}
input[type="checkbox"]:checked {
  width: 20px;
  height: 20px;
}
.newsListBox {
  display: flex;
  flex-direction: row;
  justify-content: center;
  width: 85%;
  padding-top: 40px;
  margin-bottom: 200px;
  margin-left: auto;
  margin-right: auto;
}
.categoryBox {
  display: flex;
  flex-direction: column;
  border-right: 1px solid #d6dbe5;
  margin-right: 35px;
  padding-right: 25px;
}
.categoryTitle {
  white-space: nowrap;
  color: #d6dbe5;
  text-align: center;
  font-size: 30px;
  font-weight: 700;
  margin: 3vh 0;
  cursor: pointer;
}
.activeCategory {
  color: #009fe8;
}
.unselected {
  color: rgb(181, 181, 181);
}
.unselected:hover {
  color: rgb(136, 136, 136);
}
.dropdownBox {
  font-size: 1.5em;
  margin-bottom: 50px;
  display: flex;
  flex-direction: column;
}
.dropdown {
  position: relative;
  display: inline-block;
  cursor: pointer;
  width: 100%;
}

.dropdown-list {
  position: absolute;
  top: 100%;
  left: 0;
  border: 1px solid #aaa;
  max-height: 200px; /* Set the max height for the dropdown */
  overflow-y: auto; /* Enable vertical scrolling if needed */
}

.list-item {
  display: flex;
  justify-content: left;
  align-items: center;
  width: 75vw;
  padding: 10px;
  border-bottom: 1px solid #aaa;
  white-space: nowrap;
}

.item-title,
.item-link {
  overflow: hidden;
  text-overflow: ellipsis;

  width: 55vw;
}

input[type="checkbox"] {
  margin-right: 30px;
}

.newscontain {
  padding: 3% 0%;
  display: flex;
  justify-content: flex-start;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 50px;
}
.page-number {
  cursor: pointer;
  margin: 0 10px;
  font-size: 22px; /* 원하는 폰트 크기로 조정하세요 */
  color: #919191;
}
.dateSetBtn {
  margin: 1em;
  cursor: pointer;
  font-size: 1em;
  border: 1px solid #5f5f5f;
  border-radius: 50px;
  color: #5f5f5f;
  background-color: #ffffff;
  text-align: center;
  width: 100px;
  height: 5vh;
}
.current-page {
  font-size: 27px; /* 현재 페이지 폰트 크기를 키우는 부분입니다. */
  font-weight: bold;
  color: black;
}
.prevBtn {
  cursor: pointer;
  margin-right: 10px;
}
.nextBtn {
  cursor: pointer;
  margin-left: 10px;
}
</style>
