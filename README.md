# rxForBeginner

### rxSwift에 대한 기초

## Rx란

비동기 데이터 스트림을 처리하기 위한 API

2007년, MS의 인터넷 제품 및 서비스 연구 조직인 Live lab에서 Volta라는 다양한 기술을 한 플랫폼에서 실행할 수 있는 환경을 제공함으로써 기술 습득 비용을 줄이는 프로젝트를 진행했는데, 이게 중단되고 개발팀이 Reactive Framework 라는 프로젝트로 분리되었고 이후 계속 개발된 것이 rx이다.

처음엔 rx.NET이 첫 릴리스 되었고, 2012년 오픈소스화 되면서 rxJava(넷플릭스에서 제작) 등이 공개되었다.

## Observable

observer가 구독을 가능하게 만들어주는 형태.
Observable이란 스레드 내에서 작동하는 하나의 작업 단위임.
Observable이 가지고 있는 스트림은 유한일 수도 무한일 수도 있음.

Observable은 동기/비동기, 블락/넌블락 등 다양한 설정이 가능함.

### Cold Observable vs Hot Observable

Cold Observable
	구독이 이루어진 시점부터 이벤트를 발산함.
	비동기적 작업을 실행함.
  주로 VOD로 비교됨.

Hot Observable
	구독과 상관없이 선언된 순간부터 이벤트를 발산함.
	동기적 작업을 실행함.
  주로 LIVE로 비교됨.
  이미 놓친 이벤트를 받을 수 없음.

## Observer

Observable의 이벤트를 받는 대상

## Scheduler

Rx가 실행되기 위한 thread를 스케듈링 해줌.

종류로는 시리얼(serial)과 컨커런트(concurrent)가 있고, 각각 동기/비동기 설정이 가능함.
다음은 목적에 따라 이미 정의된 스케쥴러.

시리얼 스케듈러 (싱글 스레드)
		- CurrentThreadScheduler : 현재 스레드에 있는 스케듈러, 기본 스케듈러
		- MainScheduler : 메인 스레드에 있는 스케듈러
		- SerialDispathQueueScheduler : 특정 큐(DispathQueue)에 있는 스케듈러

컨터런트 스케듈러 (멀티 스레드)
		- ConcurrentDispathQueueScheduler : 특정 큐(DispathQueue)에 있는 스케듈러
		- OperationQueueScheduler : 특정 큐(OperationQueue)에 있는 스케듈러

이 외에도 커스텀 스케듈러를 설정할 수 있음.

## Subject

Oberservable과 Observer의 속성을 모두 가지는 특별한 형태
이벤트를 방출할 수도 있고, 이벤트를 받을 수도 있음.

PublishSubject : 구독 이전의 이벤트는 방출하지 않고, 에러 이후의 이벤트도 방출하지 않음.
BehaviorSubject : Publish와 거의 동일하지만, 반드시 초기 값을 설정해 줘야함.
				초기값 혹은 마지막 이벤트의 방출값을 Observer에서 방출함.
ReplaySubject : 미지 정해진 만큼의 최신 이벤트를 새로운 Observer에게 방출함.

## Event

### 기본형식
onNext<T>
onError(swift.Error)
onComplete

onNext는 몇 번 불려도 크게 상관 없으나, onError나 onComplete 는 단 한 번, 둘 중 하나만 불림.

## Disposing

Observable의 메모리를 해제해주는 작업.
rxSwift에서는 disposebag을 사용함.
Disposebag이 deinit될 때, 모든 메모리를 해제함.

## single

event로 성공값과 에러만을 가지는 Observable

onSuccess
onError

두 메서드만을 가지며, 둘 중 하나만 단 한 번 호출된다.
메서드가 호출되면 생성주기는 끝나고 구독이 종료된다.

## 연산자 체인

대부분의 연산자들은 Observable 상에서 동작하고, Observable을 리턴함.
이 접근 방법은 연산자들을 연달아 호출 할 수 있는 연산자 체인을 제공한다.
연산자 체인에서 각각의 연산자는 이전 연산자가 리턴한 Observable을 기반으로 동작하며 동작에 따라 Observable을 변경한다.
빌더 패턴의 메서드 체인과 달리 Observable의 연산자 체인은 호출 순서에 따라 실행 결과가 달라진다.

## 사용자 연산자

***
extension ObservableType {
    func myMap<R>(transform: @escaping (E) -> R) -> Observable<R> {
        return Observable.create { observer in
            let subscription = self.subscribe { e in
                    switch e {
                    case .next(let value):
                        let result = transform(value)
                        observer.on(.next(result))
                    case .error(let error):
                        observer.on(.error(error))
                    case .completed:
                        observer.on(.completed)
                    }
                }
            return subscription
        }
    }
}
***

## BackPressure

소비자가 생산자에게 피드백을 주는 채널.
소비자가 데이터 처리량을 일정 수준으로 제어하여 과부하 상태에서도 소비자 또는 메시징 미들웨어가 포화되거나 응답성이 나빠지는 상황을 막을 수 있다.
내부에서 reqeust()로 관리하였으나, rxJava2에서부터 Observable과 Flowable로 분리됨.

rxSwift에서는 배압처리를 극히 특수한 경우라고 가정하고 지원하지 않은 대신, 다음 명령어를 사용하도록 권장함.

## Debounce vs #Throttle

Debounce	일정 시간을 기준으로 이벤트를 나누고 해당 그룹 중 가장 마지막 이벤트만 호출

Throttle	 	이벤트 사이에 일정 시간으로 간격을 두고 호출.

## 예외

정상 값을 얻지 못하거나, 아무런 응답을 받지 못 하는 경우를 처리해준다.

onErrorReturn 으로 에러처리를 해줌.
Timeout 으로 무응답 처리를 해줌.
retry를 통하여 원하는 횟수만큼 재시도를 해줌.



