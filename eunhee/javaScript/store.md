\*02.27.2023 클래스 컴포넌트 형식으로 짜여진 바닐라 자바스크립트 코드입니다. 리액트가 아닌 바닐라 자바스크립트에서 클래스형 컴포넌트 방식의 코드에 대한 이해도를 높이기 위함입니다.

## 바닐라JS로 짜여진 스토어 코드를 보니

상태를 관리할 때 상태를 데이터 프로퍼티가 아닌 접근자 프로퍼티로 생성하는 점이 흥미로웠습니다. 하단 예시 코드는 직접적으로 데이터를 프로퍼티값으로 저장하지 않고 [[get]] 어트리뷰트 내부에 값을 저장하고, [[set]] 을 통해 상태 변경시 외부 함수 실행 가능성을 열어둔 코드라고 생각했습니다. 그래서 상태를 공유하는 Message 클래스 컴포넌트의 constructor 에서 subscribe 함수를 통해 상태에 관련 외부함수(리렌더링을 위한 등) 를 observers 객체에 저장하고 해당 상태가 변경되면 외부함수가 실행할 수 있도록 설계한 점이 인상깊었습니다.

아직 코드상에서 이해되지 않는 부분은 1. 상태 변경하는 부분에 this.state[key]= val 가 아닌 state[key]= val 로 작성된 부분입니다. this.state[key]= val 를 해야 store 내부에서 관리되는 state 내부에서 변경되는 게 아닌가란 생각을 했는데 아직 this 바인딩에 대한 이해가 부족하다고 느껴 this 바인딩에 대한 공부의 필요성을 느꼈습니다. 2. 접근자 프로퍼티는 자신이 직접 데이터를 저장해 가지고 있는 게 아니라 다른 데이터 프로퍼티를 접근한다라고 알고 있는데 해당 값을 직접 조회 변경할 수 있는 점이 의아했습니다. messageStore.state.message =changedValue

```js
//스토어 내부 코드
Class Store {
constructor(state){
	this.state = {}
	this.observers = {}  //상태 변경시 추가로 실행할 함수 저장

	for(const key in state){
	  Object.definedProperty(this.state, key,{
		get: ()=>state[key],
		set: val =>{
			state[key]= val  //상태 변경
			this.observers[key]()
            //상태 변경시 상태를 공유하는 외부 컴포넌트도 상태반영을 위해 ,혹은 상태 변경시 외부함수를 실행시키기위해
      		}
	})
	}
}

subscribe(key,cb){ //cb 는 callback 함수를 나타낸다.
    // observers= {message: ()=>{}}
	this.observers[key]= cb
}
}
```

```js
//스토어 초깃값
const messageStore =  new Store({
message: ‘Hello~’
})

//스토어 상태 조회
messageStore.state.message

//스토어 상태 변경
messageStore.state.message =changedValue

//상태 변경시 상태를 공유하는 다른 컴포넌트 리렌더링
Export default class Message 컴포넌트 extends 뫄뫄 {
constructor(){
	super()
    //subscribe 함수를 실행시켜 store 내부 observers 객체에 message 프로퍼티 값으로
    //()=>{this.render()}를 저장한다.
	messageStore.subscribe(‘message’,()=>{
 	this.render()
 })
}
render(){
	this.el.innerHTML = /*html*/`
	  <h2> ${messageStore.state.message}</h2>
`
}
}
```

### 상태를 여러 컴포넌트에서 조회하고 있을 경우 (외부 함수(effect)가 여러 개일 경우 )

```js
//set 어트리뷰트 설정시
 set: val => {
          state[key] = val
          if (Array.isArray(this.observers[key])) { // 호출할 콜백이 있는 경우!
            this.observers[key].forEach(observer => observer(val))
          }
        }



//구독 함수
subscribe(key, cb) {
    Array.isArray(this.observers[key]) // 이미 등록된 콜백이 있는지 확인!
      ? this.observers[key].push(cb) // 있으면 새로운 콜백 밀어넣기!
      : this.observers[key] = [cb] // 없으면 콜백 배열로 할당!

    // 예시)
    // observers = {
    //   구독할상태이름: [실행할콜백1, 실행할콜백2]
    //   movies: [cb, cb, cb],
    //   message: [cb]
    // }
  }
```
