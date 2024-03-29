# VUEX 사용

![image-20220805114511959](assets/image-20220805114511959.png)

매장 폐기를 할 경우 해당 매장의 이미지와 해당 매장의 이름을 받아와야한다. 

그러나 내가 해당 매장의 정보를 가지고 있는 것은 MypageOwnerView.vue이고

해당 매장의 정보를 보여주는 곳은 App.vue이기 때문에 vuex를 사용하는 것이 적절하다!



따라서 vuex를 공부해봐야겠다



## VUEX란

* 정의
  * 애플리케이션의 모든 컴포넌트에 대한 중앙 집중식 저장소 역할을 하며 예측 가능한 방식으로 상태를 변경 가능

* VUEX의 패턴

![image-20220805115037182](assets/image-20220805115037182.png)

```
[State]

Vue component에서는 원본 소스 역할인 data로 볼 수 있습니다.

state는 mutation을 통해서만 변경이 가능합니다.


[Mutations]

유일하게 state를 변경할 수 있는 방법이며 메서드와 유사합니다.

commit을 통해서만 호출 할 수 있으며 함수로 구현됩니다.

API를 통해 전달받은 데이터를 mutations에서 가공하여 state를 변경합니다.


[Actions]

비동기 작업이 가능합니다.

mutation을 호출하기 위한 commit이 가능합니다.

action은 dispatch를 통해 호출할 수 있습니다.

axios를 통한 api 호출과 그 결과에 대해 return을 하거나 mutation으로 commit하는 용도로 사용됩니다.


[Getter]

Vue componet의 computed처럼 계산된 속성입니다.

state에 대해 연산을 하고 그 결과를 view에 바인딩 할 수 있습니다.

state의 변경 여부에 따라 view를 업데이트합니다.
```



* vuex의 흐름
  * vue component에서 dispatch를 통해 action 실행
  * action에서는 외부 api를 호출하는 등 비동기 로직 처리
    * 그 결과를 이용해 동기 로직인 mutations를 호출
  * mutation에서 state를 변경
    * state(data)를 변경할 수 있는 것은 오로지 mutations뿐!
    * filter, reduce 등 다양한 방법으로 데이터 가공 가능
  * getter를 이용하여 다시 component에 바인딩하면 갱신



## 직접 코딩

1. js 만들기 `src/store/modules/discardStore.js`

   ```js
   const discardStore = {
     // namespaced를 true로 해야 module namespace error가 안뜸!
     namespaced: true,
     
     // state 가지고 오고 싶은 변수 이름 설정
     state: {
       discardStoreId: "",
       discardStoreImg: "",
       discardStoreName: "",
     },
     // getters: 받아올 데이터 state와 연결
     getters: {
       discardStoreId: (state) => state.discardStoreId,
       discardStoreImg: (state) => state.discardStoreImg,
       discardStoreName: (state) => state.discardStoreName,
     },
     // actions 원하는 변수를 받아올 함수 지정
     actions: {
       discardStoreId: ({ commit }, storeId) => {
         commit("DISCARD_STOREID", storeId);
       },
       discardStoreImg: ({ commit }, storeImg) => {
         commit("DISCARD_STOREIMG", storeImg);
       },
       discardStoreName: ({ commit }, storeName) => {
         commit("DISCARD_STORENAME", storeName);
       },
     },
     // mutations 설정한 변수에 원하는 데이터 넣기
     mutations: {
       DISCARD_STOREID: (state, storeId) => {
         state.discardStoreId = storeId;
       },
       DISCARD_STOREIMG: (state, storeImg) => {
         state.discardStoreImg = storeImg;
       },
       DISCARD_STORENAME: (state, storeName) => {
         state.discardStoreName = storeName;
       },
     },
   };
   
   export default discardStore;
   ```

2. index.js에 등록하기 `src/store/index.js`

   ```js
   import Vue from "vue";
   import Vuex from "vuex";
   import createPersistedState from "vuex-persistedstate"
   
   Vue.use(Vuex);
   
   import discardStore from "@/store/modules/discardStore";
   export default new Vuex.Store({
     plugins: [createPersistedState()],
     modules: {
       discardStore,
     },
   });
   ```

3. 보낼 데이터가 있는 곳 `src/views/profile/MypageOwnerView.vue`

   ```vue
   <script>
   import {mapActions} from "vuex";
   export default {
       name:"MypageOwnerView",
       methods: {
           ...mapActions("discardStore", ["discardStoreId", "discardStoreName", "discardStoreImg",]),
           selectStore() {
               //원래 정의된 this.storeId를 discardStoreId에 넣어놓음
               // this.storeId를 불러오는 과정은 생략
               this.discardStoreId(this.storeId);
         		this.discardStoreName(this.storeName);
         		this.discardStoreImg(this.storeImg);
           }
       }
   }
   </script>
   ```

4. 데이터를 받아오는 곳 `src/components/profile/StoreInfoDiscardModal.vue`

   ```vue
   <template>
   	<div>
           <span>{{ this.discardStoreId }}</span>
           <span>{{ this.discardStoreName }}</span>
           <span>{{ this.discardStoreImg }}</span>        
       </div>
   </template>
   
   <script>
   import { mapGetters } from "vuex";
   export default {
       name: "NoticeModal",
       //computed해서 mapgetters를 불러와야함!
       computed: {
           ...mapGetters("discardStore", [
             "discardStoreId",
             "discardStoreName",
             "discardStoreImg",
           ]),
         },
   </script>
   ```

   이렇게 하면 discardStoreId, discardStoreName, discardStoreImg 변수 사용 가능!!