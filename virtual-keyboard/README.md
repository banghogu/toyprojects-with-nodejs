# Virtual Keyboard
npm install 한후 package.json에 입력된 scripts를 참고하여 dev server를 실행하거나 build 하면 됩니다.

### 가상 키보드 프로젝트

<hr>

**프로젝트 기간 : 2023/12/05 ~ 2023/12/06
프로젝트 목적 : node js로 간단한 앱을 만들고 webpack으로 deploy해보기**

<hr>

### 초기 설정 
https://github.com/banghogu/toyprojects-with-nodejs/tree/main/webpack-boilerplate

보일러 플레이트를 활용하여 eslint, webpack, prettier, package json 기본설정을 해주었지만 아직 json의 의존성 설정, webpack config 설정 코드는 어떻게 연결되고 로직이 짜여져있는지 모르므로 공부가 더 필요하다.

![](https://velog.velcdn.com/images/banghogu/post/67f0af62-852c-47d2-9d9e-1ebf08dc32e2/image.png)

보일러 플레이트는 나중에 프로젝트를 제작 할 때 또 재사용 할 수 있으므로 깃허브에 폴더 형식으로 저장해서 필요할때마다 갖고와서 새로운 프로젝트를 만들 수 있도록 해주었다.

<hr>

### html, css

![](https://velog.velcdn.com/images/banghogu/post/e2c08bf7-eafb-4ff5-8dcd-78037aa6fcca/image.png)
![](https://velog.velcdn.com/images/banghogu/post/3792378e-3e7a-4d5a-a974-d1a51fe8cff7/image.png)

보일러 플레이트에 따라 css 연결은 html header태그에서 이뤄지는것이 아닌 webpack config 설정에 따라 연결시켜줬다.

<hr>

### js

**keyboard js**
```
export class Keyboard {
  #swichEl;
  #fontSelectEl;
  #containerEl;
  #keyboardEl;
  #inputGroupEl;
  #inputEl;
  #keyPress = false;
  #mouseDown = false;
  constructor() {
    this.#assignElement();
    this.#addEvent();
  }

  #assignElement() {
    this.#containerEl = document.getElementById("container");
    this.#swichEl = this.#containerEl.querySelector("#switch");
    this.#fontSelectEl = this.#containerEl.querySelector("#font");
    this.#keyboardEl = this.#containerEl.querySelector("#keyboard");
    this.#inputGroupEl = this.#containerEl.querySelector("#input-group");
    this.#inputEl = this.#inputGroupEl.querySelector("#input");
  }

  #addEvent() {
    this.#swichEl.addEventListener("change", this.#onChangeTheme);
    this.#fontSelectEl.addEventListener("change", this.#onChangeFont);
    document.addEventListener("keydown", this.#onKeyDown.bind(this));
    document.addEventListener("keyup", this.#onKeyUp.bind(this));
    this.#inputEl.addEventListener("input", this.#onInput);
    this.#keyboardEl.addEventListener(
      "mousedown",
      this.#onMouseDown.bind(this)
    );
    document.addEventListener("mouseup", this.#onMouseUp.bind(this));
  }

  #onMouseUp(event) {
    if (this.#keyPress) return;
    this.#mouseDown = false;
    const keyEl = event.target.closest("div.key");
    const isActive = !!keyEl?.classList.contains("active");
    const val = keyEl?.dataset.val;
    if (isActive && !!val && val !== "Space" && val !== "Backspace") {
      this.#inputEl.value += val;
    }
    if (isActive && val === "Space") {
      this.#inputEl.value += " ";
    }
    if (isActive && val === "Backspace") {
      this.#inputEl.value = this.#inputEl.value.slice(0, -1);
    }
    this.#keyboardEl.querySelector(".active")?.classList.remove("active");
  }

  #onMouseDown(event) {
    if (this.#keyPress) return;
    this.#mouseDown = true;
    event.target.closest("div.key")?.classList.add("active");
  }

  #onInput(event) {
    event.target.value = event.target.value.replace(/[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]/, "");
  }

  #onKeyDown(event) {
    if (this.#mouseDown) return;
    this.#keyPress = true;
    this.#inputGroupEl.classList.toggle(
      "error",
      /[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]/.test(event.key)
    );

    this.#keyboardEl
      .querySelector(`[data-code=${event.code}]`)
      ?.classList.add("active");
  }

  #onKeyUp(event) {
    if (this.#mouseDown) return;
    this.#keyPress = false;
    this.#keyboardEl
      .querySelector(`[data-code=${event.code}]`)
      ?.classList.remove("active");
  }

  #onChangeTheme(event) {
    document.documentElement.setAttribute(
      "theme",
      event.target.checked ? "dark-mode" : ""
    );
  }

  #onChangeFont(event) {
    document.body.style.fontFamily = event.target.value;
  }
}

```

**index js**
```
import "../css/style.css";
import { Keyboard } from "./keyboard";
new Keyboard();
```


index js에서 keyboard js를 import해주어 연결시켰다. keyboard js에서는 html요소들에 맞게 변수들을 가져왔다.

<hr>

### 알게된점

keyboard js에서 리액트 클래스형 컴포넌트처럼 하나의 기능을 클래스로 만들어서 export class로 내보내주는 형식이다.

<hr>

![](https://velog.velcdn.com/images/banghogu/post/19e349dc-b15b-4b51-827a-e99aa9bfb659/image.png)
위 부분은 꼭 필요하지는 않지만 남이 내 코드를 봤을때나, 나중에 내 코드를 수정 및 확인 할 때 더 쉽게 분석하기 위해 써주는것이다. # 해쉬를 붙이는것은  private class field로 클래스 외부에서 직접 접근할 수 없게 해준다.

<hr>


getElementById는 document element에서만 쓸 수 있다. 변수에서 객체를 찾으려면 querySelector를 이용한다.

<hr>


bind(this)는 함수 바인딩으로 콜백함수에 this를 넣으면 this 정보가 사라지는 문제를 해결한다. this.#swichEl.addEventListener("change", this.#onChangeTheme);에서는 this로 접근하여 그대로 콜백함수로 this를 넣어줘서 잘 찾아지겠지만 document.addEventListener로 접근 할 시 내부 함수에서 this를 찾지 못해 함수바인딩이 필요하다


<hr>

> setAttribute(
      "theme",
      event.target.checked ? "dark-mode" : ""
    );

에서처럼 뒤에 설정값으로 삼항 연산자를 넣어줄 수 있다.

<hr>

document.body.style.fontFamily 처럼 document.body.style로 요소의 스타일을 변경 시켜줄수 있다.

> this.#inputGroupEl.classList.toggle(
      "error",
      /[ㄱ-ㅎ|ㅏ-ㅣ|가-힣]/.test(event.key)
    );
   
   
에서 inputgorup에 대하여 한글이 있을시 true가 되기 때문에 error가 들어가고 영문이 있을땐 false가 되어 error클래스가 사라진다.

<hr>


> this.#keyboardEl
      .querySelector(`[data-code=${event.code}]`)
      ?.classList.add("active");
      
    
keyboardEl에서 data-code=${event.code}인 요소가 있으면 ?.classList.add("active");로 active 클래스를 추가해준다. 이를 옵셔널 체이닝이라고 한다. 방법은 위에처럼 ?.

<hr>

Element.closest() : 현재 요소에서 가장 가까운 상위요소를 찾는다. 	

<hr>

> const isActive = !!keyEl?.classList.contains("active");

contains에서 active가 있냐라고 메소드를 통해 undefined, true 형식으로 내놓을텐데 좀 더 확실한 boolean 값을 위해 !!를 사용한다
