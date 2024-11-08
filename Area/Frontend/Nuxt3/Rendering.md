## SSR
---
서버에서 HTML을 렌더링하여 초기 로드가 빠르고 SEO에 유리하지만 서버 부하가 증가할 수 있다.
서버로 요청이 오면 서버에서 JavaScript (Vue.js) 코드를 실행 시켜서 HTML 을 만들어낸 뒤 응답한다. (기존 SSR 방식) 그리고 응답을 받은 브라우저는 HTML 을 다운 받은 뒤 서버에서 실행시켰던 JavaScript 를 받은 후 실행한다. 그 뒤 Vue.js 는 해당 문서에 대한 제어를 시작하게 되고 상호작용 할 수 있게 된다. 이 과정을 **Hydration** 이라고 한다.
![[Pasted image 20241031154119.png]]
## CSR
---
클라이언트에서 JavaScript로 페이지를 렌더링하여 초기 로드가 느릴 수 있지만 이후 페이지 전환이 빠르고 부드럽습니다.

## Universal Rendering(default)
---
브라우저가 범용 렌더링이 활성화된 URL을 요청하면 Nuxt는 서버 환경에서 JavaScript(Vue.js) 코드를 실행하고 완전히 렌더링된 HTML 페이지를 브라우저에 반환합니다. Nuxt는 페이지가 미리 생성된 경우 캐시에서 완전히 렌더링된 HTML 페이지를 반환할 수도 있습니다. 사용자는 클라이언트 측 렌더링과 달리 애플리케이션의 초기 콘텐츠 전체를 즉시 얻습니다.
```vue
<script setup lang="ts">
const counter = ref(0); // executes in server and client environments

const handleClick = () => {
  counter.value++; // executes only in a client environment
};
</script>

<template>
  <div>
    <p>Count: {{ counter }}</p>
    <button @click="handleClick">Increment</button>
  </div>
</template>

```

#### http 통신의 중복호출
ofetch 라이브러리는 환경에 따라 Server면 node-fetch-native, Browser면 브라우저의 기본 fetch를 선택해 사용하고 $fetch 메서드를 사용한다.
그러나 server에서 만든 html과 client에서 렌더링된 html의 결과가 다를때 hydration mismatch warning이 뜨게 된다. 그 뿐만 아니라 server는 불필요하게 두번의 api호출을 받게 되므로 서버 성능의 저하를 일으킬 수 있다. > useFetch와 useAsyncData의 등장
[[API 호출]]
