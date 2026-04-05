# Reflow와 Repaint 정리

웹 페이지가 화면에 보이기까지 브라우저는 DOM, CSSOM을 바탕으로 렌더 트리를 만들고, 레이아웃을 계산한 뒤 화면에 그립니다.  
이 과정에서 자주 등장하는 개념이 **Reflow**와 **Repaint**입니다.

---

## 1. Reflow란?

**Reflow**는 브라우저가 **요소의 크기, 위치 등 레이아웃을 다시 계산하는 작업**입니다.

즉, 화면 내에서  
- 어떤 요소가 어디에 배치되는지
- 얼마나 넓고 높은지
- 다른 요소에 어떤 영향을 주는지

를 다시 계산하는 과정입니다.

### Reflow가 발생하는 경우
- DOM 요소 추가 / 삭제
- 요소의 `width`, `height` 변경
- `margin`, `padding`, `border` 변경
- 폰트 크기 변경
- 브라우저 창 크기 변경
- `display` 변경

### 예시

```css
.box {
  width: 200px;
}
```

여기서 width 값을 바꾸면 해당 요소의 크기가 달라지고, 주변 요소의 배치에도 영향을 줄 수 있으므로 Reflow가 발생합니다.

특징
레이아웃을 다시 계산해야 해서 비용이 큼
변경된 요소뿐 아니라 부모 / 자식 / 형제 요소까지 영향이 퍼질 수 있음
성능 저하의 주요 원인이 될 수 있음

## 2. Repaint란?

Repaint는 레이아웃 변화 없이 요소의 시각적인 부분만 다시 그리는 작업입니다.

즉, 위치나 크기는 그대로이고

색상
배경
그림자
테두리 색

같은 외형만 바뀌는 경우 발생합니다.

Repaint가 발생하는 경우
color 변경
background-color 변경
visibility 변경
box-shadow 변경

예시
```css
.box {
  background-color: blue;
}
```

여기서 배경색만 바꾸면 요소의 위치나 크기는 변하지 않으므로 Reflow 없이 Repaint만 발생합니다.

특징
레이아웃 계산은 하지 않음
Reflow보다 비용은 적지만, 자주 발생하면 성능에 영향이 있음

## 3. Reflow와 Repaint의 차이

| 구분    | Reflow                         | Repaint                        |
| ----- | ------------------------------ | ------------------------------ |
| 의미    | 레이아웃을 다시 계산                    | 화면을 다시 그림                      |
| 영향 범위 | 크기, 위치, 배치 변화                  | 색상, 배경 등 외형 변화                 |
| 비용    | 큼                              | 비교적 작음                         |
| 예시    | `width`, `height`, `margin` 변경 | `color`, `background-color` 변경 |


한 줄 정리
Reflow: 배치를 다시 계산하는 것
Repaint: 계산된 결과를 다시 그리는 것

또한 보통 Reflow가 발생하면 Repaint도 함께 발생하는 경우가 많습니다.

## 4. 왜 중요한가?

프론트엔드 성능 최적화에서 Reflow와 Repaint를 이해하는 것은 매우 중요합니다.

특히 Reflow는 비용이 큰 작업이라, 자주 발생하면

화면 끊김
애니메이션 버벅임
렌더링 지연

같은 문제가 생길 수 있습니다.

## 5. 최적화 방법

### 5-1. Reflow를 유발하는 속성 변경 최소화

레이아웃에 영향을 주는 속성은 Reflow를 발생시키므로 자주 변경하지 않는 것이 좋습니다.

주의할 속성 예시
width
height
margin
padding
border
top
left
좋은 방향
스타일 변경을 여러 번 나누지 말고 한 번에 처리하기
잦은 레이아웃 변경 대신 클래스 토글 활용하기

```javascript
// 비효율적: 여러 번 스타일 변경
element.style.width = "200px";
element.style.height = "100px";
element.style.margin = "20px";

// 상대적으로 좋음: 클래스 한 번 변경
element.classList.add("active");
```

### 5-2. 애니메이션은 transform, opacity 중심으로 처리하기

애니메이션에서 width, height, top, left 등을 변경하면 Reflow가 발생할 수 있습니다.
반면 transform, opacity는 레이아웃 변경 없이 처리될 가능성이 높아 성능에 더 유리합니다.

비추천
```css
.box {
  position: relative;
  left: 100px;
}
```

추천
```css
.box {
  transform: translateX(100px);
}
```

이유
transform: 위치를 시각적으로만 변경
opacity: 투명도만 변경

이 둘은 일반적으로 애니메이션 성능 최적화에 자주 사용됩니다.

### 5-3. will-change는 필요한 곳에만 사용하기

will-change는 브라우저에게
“이 요소는 곧 이런 속성이 바뀔 거야”
라고 미리 알려주는 속성입니다.

```css
.box {
  will-change: transform;
}
```

장점
- 브라우저가 사전 준비를 할 수 있어 애니메이션이 더 부드러워질 수 있음

주의점
- 너무 많은 요소에 사용하면 오히려 메모리 사용량이 늘어날 수 있음
- 정말 변화가 자주 일어나는 요소에만 제한적으로 사용해야 함

## 6. 실무에서 기억할 포인트
- Reflow는 레이아웃 재계산이라 비용이 크다.
- Repaint는 다시 그리는 작업이라 Reflow보다는 가볍다.
- 애니메이션은 가능하면 transform, opacity를 사용한다.
- 레이아웃 변경이 많은 코드는 묶어서 처리한다.
- will-change는 남용하지 않는다.


## 7. 정리

> Reflow는 요소의 크기와 위치를 다시 계산하는 과정이고,
> Repaint는 계산된 결과를 바탕으로 화면에 다시 그리는 과정입니다.

> 성능 최적화를 위해서는 Reflow 발생을 최대한 줄이고,
> 애니메이션이나 시각적 변화는 transform, opacity 위주로 처리하는 것이 좋습니다.
