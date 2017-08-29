# std::shared_ptr 처럼 작동하되 대상을 잃을 수도 있는 포인터가 필요하면 std::weak_ptr를 사용하라

#### weak_ptr의 특징
1. std::shared_ptr로 부터 생성된다. 이때 weak_ptr은 shared_ptr과 동일한 객체를 가리키지만 
   참조 횟수에는 영향을 주지 않는다.
2. weak_ptr이 가리키는 객체가 해제되었는지 확인할 수 있다.


