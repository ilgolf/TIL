# Svelte에서 사용하는 반응형 구문

## 반응형 구문이란? 

특정 조건이나 상황에 따라 다르게 동작하는 코드 구문을 말한다. 프로그래밍에서는 주로 이벤트나 상황 변화에 따라 동적으로 동작하는 코드를 작성할 때 사용한다.

Javascript 이벤트 리스터가 대표적이다.

```javascript
document.getElementById("myButton").addEventListener("click", function() {
    alert("Button was clicked!");
});
```

## Svelte에서의 반응형 구문

Svelte에서도 반응형 구문을 제공해준다. $ 구문으로 표현되며 $ 구문은 반응형 구문으로 구성 요소가 변경되면 DOM을 자동으로 업데이트 한다. 
실제 구현 코드를 보면 다음과 같다.

```svelte
<script>
	let count = 0;
	function increment() {
		count += 1;
	}
	$: doubled = count * 2;
</script>
<button on:click={increment}>
	Clicked {count}
	{count === 1 ? 'time' : 'times'}
</button>
<h2>클릭 횟수 * 2</h2>
<p>{doubled}</p>
```

## 반응형 구문 사용시 주의 사항

반응형 구문은 구성 요소에 따라 무조건 DOM 업데이트가 일어나기 때문에 이 점을 유의하여 사용해야한다. 만일 검색 동작이 반응형 구문으로 동작하고 구성 요소가 DOM 변경에 영향을 주고 있다면
원하는 방향으로 동작하지 않을 수 있다. (구성 요소에 따라 자동으로 검색 요청이 서버로 갈 수도 있다.)