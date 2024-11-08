### $fetch
$fetch 같은 경우에는 Nuxt3 에서 제공하는 전역 메서드이다. 해당 메서드를 통해 HTTP request 를 만들 수 있다. $fetch 는 [ofetch](https://github.com/unjs/ofetch) 라는 라이브러리를 사용하고 있는데 해당 라이브러리는 Node 환경과 Browser 환경 두군데서 같이 사용할 수 있다고 한다. 자동으로 실행환경에 따라 Node 환경에서 실행된다면 [node-fetch-native](https://github.com/unjs/node-fetch-native), Browser 환경에서 실행된다면 Browser 의 [fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 를 사용한다고 한다. Nuxt3 가 국민 라이브러리인 Axios 를 채택하지 않은 이유는 [여기서](https://github.com/nuxt/nuxt/discussions/16605#discussioncomment-2616956) 찾아볼 수 있다.
```vue
<template>
    <div>
        <button @click="testFetch">testFetch</button>
        <div>data : {{ data }}</div>
    </div>
</template>
<script setup lang="ts">
const data = ref();
const testFetch = async () => {
    const result = await $fetch<number>("/api/temp");
    data.value = result;
};
</script>
```
$fetch 의 파라미터
**url** : 요청 url
**options**:
- **method** : 요청 메서드 설정
- **query** : URL 에 붙일 query params. [ufo](https://github.com/unjs/ufo) 라이브러리를 통해 만들어진다.
- **params** : query 와 똑같다.
- **body** : 요청 body
- **headers** : 요청 headers
- **baseURL** : 요청 baseURL
$fetch 는 기본적으로 Client side 에서만 사용해야한다. 서버와 클라이언트환경에서의 중복 호출의 문제가 있기 때문이다. 즉 Top level await 를 통한 SSR 시 $fetch 를 통해 data 를 가져오면 안된다는 뜻이다. 그것을 해결하기 위해 Nuxt3 에서 제공하는것이 useAsyncData 와 useFetch 이다.
### useAsyncData
useAsyncData([문서링크](https://nuxt.com/docs/api/composables/use-async-data)) 는 Nuxt3 에서 제공하는 전역 메서드이다. **하지만!** 여기에는 아주 중요한 한가지 예외가 있는데 이 메서드는 setup function, plugin, 또는 route middleware 에서만 사용이 가능하다.
```vue
<template>
    <div>
        <button @click="() => refresh()">testUseAsyncData</button>
        <div>useAsyncDataResult : {{ data }}</div>
    </div>
</template>
<script setup lang="ts">
const { data, refresh } = await useAsyncData<number>("useAsyncDataTest", () => $fetch("/api/temp"));
console.log(data.value);
</script>
```
useAsyncData 의 파라미터
**key** : 해당 비동기 메서드의 유니크한 key 값을 설정한다. 해당 key 를 통해 server 와 client 두번의 호출을 방지한다. key 특정해서 넘기지 않으면 자동으로 useAsyncData가 사용된 파일과 라인넘버의 해싱값으로 만들어 낸다.
**handler** : 실행할 비동기 함수
**options** : 
-   **lazy** : useAsyncData 의 결과를 기다리지 않고 먼저 화면을 렌더링한다.  default - false
-   **default** : 함수를 통해 비동기 함수가 완료되기 전에 보여줄 값 셋팅할 수 있다. 
-   **server** : useAsyncData 실행을 server side 에서 실행할 여부 default - true
-   **transform** : 함수를 통해 비동기 함수 실행 결과를 변형시킨다.
-   **pick** : 비동기 함수 실행결과에서 특정 key 값을 가진 data 만 return 시킨다. 
-   **watch** : 특정 값이 변경될때 자동으로 useAsyncData 를 재호출한다.
-   **immediate** : useAsyncData 를 페이지 진입시 바로 호출한다. default - true
useAsyncData 의 Return value
**data** : 비동기 함수의 결과가 담긴다. **(반응성이 있는 ref 객체이다.)**
**pending** : 비동기 함수가 실행 중인지 여부를 boolean 으로 반환한다.
**refresh**/**execute** : 내부 구현체를 보면 이 둘은 완전히 같은 함수이다. 완전 같은 일을 하는 메서드를 왜 두개 제공하는걸까? [직접 물어봤다.](https://github.com/nuxt/nuxt/discussions/20885) 결론은 immediate 옵션을 통해 data 를 바로 fetching 할지 말지 결정할 수 있는데 그때의 상황에 따라 메서드 명으로 의미있게 사용하라고 한다. (사실 그다지 필요해보이지는 않는다.)
**error** : 비동기 함수의 실행이 실패한 경우 실패 object 가 담긴다.
**status** : 현재 비동기 메서드의 상태를 반환한다. (idle - 대기중, pending - 실행중, success - 성공, error - 실패)

useAsyncData 같은 경우 script setup 안에서 Top level await 를 사용해도 중복호출이 일어나지 않는다. 중복호출을 막는것의 핵심은 key 이다. 페이지 첫 진입시 SSR 을 통해 HTML 을 만들어내는데 useAsyncData 가 실행 될 경우 Server side 에서 handler(비동기 함수)를 먼저 실행한다. 실행 된 결과값을 캐싱해 Client side 에 넘기고, hydration 이 일어나 Client side 에서 useAsyncData 를 다시 실행하면 먼저 캐싱 Object 를 먼저 검사해 해당 key 값의 value 가 있으면 handler 를 호출 하지 않고 그 value 를 사용한다. 그 덕분에 두번의 API 호출이 일어나지 않게 되는것이다.
### useFetch
useFetch([문서 링크](https://nuxt.com/docs/api/composables/use-fetch))는 위의 useAsyncData 를 좀 더 간결하게 쓰기위한 wrapper 메서드이다. 마찬가지로 전역 메서드이고 useAsyncData 가 내부적으로 사용되기때문에 setup function, plugin, 또는 route middleware 에서만 사용이 가능하다.

```vue
<template>
    <div>
        <button @click="() => refresh()">testUseFetch</button>
        <div>useFetchResult : {{ data }}</div>
    </div>
</template>
<script setup lang="ts">
const { data, refresh } = await useFetch<number>("/api/temp", { key: "useFetchTest" });
console.log(data.value);
</script>
```

useFetch 의 파라미터
useAsyncData 의 handler 를 제외한 모든 파라미터에서 와 $fetch 의 options 파라미터 를 포함한다.

useFetch 의 Return Value
useAsyncData 와 동일하다.

useFetch 도 useAsyncData 와 같은 캐싱시스템으로 Top level await 를 사용해도 중복호출을 막아준다.
### useAsyncData vs useFetch

언뜻보면 useAsyncData 가 useFetch 에 포함되어있는 단순한 관계처럼 보이지만 사실 사용법에서 몇가지 차이가 있다.

그 중 가장 큰 차이는 **실행** **context 의 차이** 이다.

#### 실행 context 의 차이

예제 코드를 통해 알아보자.

```javascript
// useAsyncDataTest.vue
const { data, refresh } = await useAsyncData<number>("useAsyncDataTest", () => $fetch("/api/temp"));
 
// useFetchTest.vue
const { data, refresh } = await useFetch<number>("/api/temp", { key: "useFetchTest" });
```

위의 코드처럼 단순한 호출은 항상 같은 결과를 낸다.

하지만 여기 dynamic 하게 호출 url 이 바뀐다던가 요청 body 의 값이 바뀌는 코드는 어떨까?

```javascript
// useAsyncDataTest.vue
const page = ref<number>(0);
const { data, refresh } = await useAsyncData<number>("useAsyncDataTest", () =>
    $fetch("/api/temp", {
        params: { page: page.value++ },
    }),
);
    
// useFetchTest.vue
const page = ref<number>(0);
const { data, refresh } = await useFetch<number>("/api/temp", {
    key: "useFetchTest",
    params: { page: page.value++ },
});
```
useAsyncData 의 두번째 매개변수는 **콜백함수**이고 useFetch 의 두번째 매개변수는 객체에서 오는 차이다. 즉, useAsyncData 의 refresh 함수가 호출될때는 콜백함수가 호출된다. 해당 콜백함수 실행시점에 외부 값들을 참조하는것이다. 하지만 useFetch 같은 경우 초기 호출시 객체로 넘기기 때문에 refresh 호출시에 항상 해당 객체만을 이용해서 호출한다. 그래서 useFetch 는 값이 변경이 안되는것이다. Nuxt3 진영에서는 해당 현상을 **freezing** 이라고 부른다.


### 결론
**1. $fetch 는 Client side 에서만 사용해야한다.**

Nuxt3 진영에서는 $fetch 는 사용자의 상호작용에서 사용하라고 한다.

> On the other hand, when wanting to make a network request based on user interaction, $fetch is almost always the right handler to go for. ([Data fetching docs](https://nuxt.com/docs/getting-started/data-fetching#data-fetching))

즉, 유저의 입력같은 post, put, delete 등등은 $fetch 를 이용하고 페이지를 처음 렌더링할때 필요한 data 를 가져오는것은 useAsyncData 나 useFetch 를 이용하라는 뜻이다. 실제로 Nuxt v3.5.0 까지 useFetch 에서 method "get" 이외에는 type error 가 났었다.([관련 issue](https://github.com/nuxt/nuxt/issues/19077))

**2. useAsyncData 와 useFetch 는 둘의 차이점을 인식하고 사용해야한다.**

필자는 useFetch 보단 useAsyncData 를 사용한다. 자동으로 refresh 해주는것은 개발자 제어에서 벗어난다고 생각하고, 다양한 값을 원하는대로 넣어주기 어렵기 때문이다.

Nuxt3 의 Data fetching 을 사용할때 가장 주의해야할 부분은 Server side 에서 호출시 cookie 나 localStorage 를 사용할 수 없다는 점이다. 해당 부분은 Nuxt3 에서 제공해주는 useState 나 pinia 를 통해서 해결해야한다.


출처 : https://jongmin4943.tistory.com/11