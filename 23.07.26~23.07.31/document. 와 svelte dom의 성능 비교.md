# document. 와 svelte dom의 성능 비교

기존 `Modal`은 다음과 같이 여러개의 `.class`를 이용해서 확인하고 추가하고 삭제하며 `DOM`에 직접 접근하여 modal을 구현하고 있습니다. (자세한 코드를 작성하지 못한점 죄송합니다.)

```svelte
<style>
	.modal_container {
		width: 100vw;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		position: fixed;
		top: 0;
		left: 0;
		background: rgba(0, 0, 0, 0.8);
	}
	.modal {
		min-width: 16em;
		padding: 5%;
		display: flex;
		flex-direction: column;
		gap: 1em;
		border: none;
		border-radius: 8px;
		text-align: center;
		background: white;
	}
	.none {
		display: none;
	}
</style>

<button on:click={on_click_js}>Js DOM</button>

<div class="modal_container none">
  <div class="modal">
    <div>Js Modal Test</div>
  </div>
</div>

<script>
	function on_click_js() {
		const $modal_container = document.querySelector(".modal_container");

		if ($modal_container.classList.contains("none")) {
			$modal_container.classList.remove("none");
		} else {
			$modal_container.classList.add("none");
		}
	}
</script>
```

그러나 이번에`svelte`로 개발을 진행하면서 `.class` 를 확인 추가 삭제하는 형식이 아닌 `svelte`로 `component`를 만들어 구현하게 되었습니다.

[ Modal.svelte ]

```svelte
<style>
	.div_background_cls {
		width: 100vw;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		position: fixed;
		top: 0;
		left: 0;
		background: rgba(0, 0, 0, 0.8);
	}

	.div_modal_cls {
		min-width: 16em;
		padding: 5%;
		display: flex;
		flex-direction: column;
		gap: 1em;
		border: none;
		border-radius: 8px;
		text-align: center;
		background: white;
	}
</style>

{#if show_modal}
	<div on:click|self={() => (show_modal = !show_modal)} class="modal_background">
		<div class="modal">
      <slot name="body" />
			<slot name="button" />
    </div>
	</div>
{/if}

<script>
	export let show_modal = false; // boolean
</script>

```

[ main.svelte ]

```svelte

  <button on:click={on_click_svelte}>Svelte DOM</button>

	<Modal bind:show_modal>
		<div slot="body">Svelte Modal Test</div>
	</Modal>

  <script>
	import Modal from "./components/Modal.svelte";

	let show_modal = false;

	function on_click_svelte() {
		show_modal = !show_modal;
	}
  </script>
```

기존의 것들이 많이 포진되어 있어 바꾸는데 리소스가 많이 소비되지만 성능, 코드의 간결성등 눈에 보이는 이점이 있으면 바꾸기로 하여 조사를 진행하게 되었습니다.

우선 저의 생각에 대해 정의를 해보면 다음과 같습니다.

계속적으로 `DOM`에 직접 접근하고 `.classList`를 통해 `.class`를 찾고 `.class`를 추가하고 제거하는 것보다 `svelte`의 DOM사용을 위해 modal을 제어하는 것이 성능적, 코드적 측면에서 좋을 것 같았습니다.

### 1. 코드적

우선 코드의 양의 측면에서 보면 비슷하지만 기존 `.class`를 주로 이용한 modal같은 경우 너무 많은 쪼갬이 있습니다.
이로 인해 제어할 것이 많아 코드의 복잡도가 높아져 코드를 파악하기 쉽지 않을 수 있습니다.

=> **양은 비슷하나 복잡도는 낮아진다.**

### 2. 성능

**DOM을 접근하는 Js DOM**과 **Svelte를 이용해서 DOM에 접근**하는 **성능을 비교**하고자 다음과 같은 화면을 화면을 만들게 되었습니다.

Modal이 굉장히 작은 조각이기 때문에 최악의 상황에서 Modal을 실행해보기 위해 Performance(Chrome 개발자 도구)에서 `CPU: 6xslowdown`과 `Network: Slow 3G`로 테스트를 진행하게 되었습니다. 또한 처음 Page loading을 할때 관련된 폴더등을 가져와서 실행하기 때문에 각각 코드를 짜서 실행하였습니다.

![성능2_gif](https://github.com/GeonWooPaeng/GeonWooPaeng/assets/53526987/08cc23a6-e52e-4b95-a99d-3b540334cb14)

다음과 같이 성능을 측정하면 다음과 같은 막대 그래프를 볼 수 있습니다.

![js1](https://github.com/GeonWooPaeng/GeonWooPaeng/assets/53526987/e0e5c878-d543-4e9f-a5ef-81021512a60a)

그런데 사실 이 부분을 보는 것 보다 **보라색 부분**이 **버튼 클릭 후 실행되는 과정을 보여주는 곳**이므로 보라색 부분을 파악해야 합니다. (svelte도 마찬가지 입니다.)

### Js

[ 테스트 코드 ]

```svelte
<style>
	/* js, svelte 버튼 모양 */
	.container {
		...
	}
	.container > button {
		...
	}
	.container > button:hover {
		...
	}

	/* modal */
	.modal_container {
		width: 100vw;
		height: 100vh;
		display: flex;
		justify-content: center;
		align-items: center;
		position: fixed;
		top: 0;
		left: 0;
		background: rgba(0, 0, 0, 0.8);
	}
	.modal {
		min-width: 16em;
		padding: 5%;
		display: flex;
		flex-direction: column;
		gap: 1em;
		border: none;
		border-radius: 8px;
		text-align: center;
		background: white;
	}
	.none {
		display: none;
	}
</style>

<div class="container">
	<button on:click={on_click_js}>Js DOM</button>

	<!-- Js Modal -->
	<div class="modal_container none">
		<div class="modal">
			<div>Js Modal Test</div>
		</div>
	</div>

</div>

<script>
	function on_click_js() {
    set_js();
	}

	function set_js() {
		const $modal_container = document.querySelector(".modal_container");

		if ($modal_container.classList.contains("none")) {
			$modal_container.classList.remove("none");
		} else {
			$modal_container.classList.add("none");
		}
	}
</script>

```

위의 코드를 돌려본 후 보라색 부분을 자세히 보면 다음과 같은 단순한 막대 그래프를 볼 수 있습니다. (약 0.006s)

![js2](https://github.com/GeonWooPaeng/GeonWooPaeng/assets/53526987/0bc1d93b-0361-4dcc-92eb-3f164c418b0d)

### Svelte

[ 테스트 코드 ]

```svelte
<style>
	/* js, svelte 버튼 모양 */
	.container {
    ...
	}
	.container > button {
		...
	}
	.container > button:hover {
		...
	}
</style>

<div class="container">
	<button on:click={on_click_svelte}>Svelte DOM</button>

	<!-- Svelte Modal -->
	<Modal bind:show_modal>
		<div slot="body">Svelte Modal Test</div>
	</Modal>
</div>

<script>
	import Modal from "./components/Modal.svelte";

	let show_modal = false;

	function on_click_svelte() {
		show_modal = !show_modal;
	}
</script>

```

위의 코드를 돌려본 후 보라색 부분을 자세히 보면 Js에 추가적인 부분이 있는 것을 볼 수 있습니다.

![svelte1](https://github.com/GeonWooPaeng/GeonWooPaeng/assets/53526987/66078185-092f-47d0-9576-d8b9d5023d0e)

해당 부분은 0.01s 정도밖에 걸리지 않습니다.

### 성능 결론

Js와 Svelte를 비교를 해보면 **Js는 약 0.005sec** **Svelte는 약 0.01sec**이 걸리는 것을 파악할 수 있었습니다.
예상과는 다르게 Js가 더 빠른 것을 파악할 수 있었습니다.

이유를 더 생각해보니 **Svelte**는 `.svelte`를 컴파일하여 **Js**로 변환하여 실행을 하게 됩니다.

위의 보라색 막대 그래프를 보면 **Js**는 **Svelte**와는 다르게 `Compile Code`를 한 후 `style`를 제정의 하고 `Layout` 진행하고 마무리를 진행과정이 나와있고 **Svelte**는 **Js**가 가지고 있는 것에 추가로 **Svelte**내용을 가지고 있는 것을 파악할 수 있습니다.

**Svelte**이 동작하는 과정에 대한 이해가 부족하고 **React(Virtual DOM)**과 비슷하게 생각했던 점이 패인인것 같습니다.

그리고 Modal component화 하는 것에 대해 생각을 하여 나눴지만 공통화하는 것에 대한 차이가 있었고 컴포넌트를 추가하는 것보다 기존의 alert, modal을 바꿔서 사용하는 것이 더 효율적이라는 의견이 많아 큰틀을 맞춰 개발을 진행하게 되었습니다.

또한 디자이너분과 협업을 진행하고 있는데 저의 범위에 맞춰서 개발을 진행하고 질문을 드렸지만 범위가 저와 디자이너분과 잘 맞지 않다는 것을 깨달았고 오히려 일이 늘어나는 상황이 발생하였습니다.
그래서 유연하게 최소한의 범위부터 점차 늘려가며 범위에 대한 협의를 해야 겠습니다. (개발자 팀원들과도 마찬가지...)

재미있는 경험들을 하게 되었고 **Svetle**의 성능을 좋게 만들기 위해서는 build, Compile 시간을 줄이는 식으로 하는 것이 더욱 효과적일 것이라는 생각도 하게 되었고 build, Compile 시간을 줄여 성능을 최적화 해봐야 겠습니다.

### Ref

https://codingmoondoll.tistory.com/entry/%ED%81%AC%EB%A1%AC-%EA%B0%9C%EB%B0%9C%EC%9E%90-%EB%8F%84%EA%B5%AC%EC%9D%98-Performance-%ED%83%AD-%EB%8B%A4%EB%A3%A8%EA%B8%B0-%EA%B8%B0%EB%B3%B8%ED%8E%B8
https://velog.io/@9wan6zae/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EC%84%B1%EB%8A%A5-%EC%B8%A1%EC%A0%95-p6u9levf
https://medium.com/myrealtrip-product/fe-website-perf-part2-e0c7462ef822
https://lihautan.com/the-svelte-compiler-handbook/#dom-renderer
