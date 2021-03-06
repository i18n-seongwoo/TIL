# Single Table Inheritance
* PEAA에 언급된 패턴이다. 
* 1개 테이블을 이용해서 상속 구조를 만드는걸 말한다.
* 말은 그럴듯하지만 개발자라면 흔히 본 내용일 것이다.
* 테이블에 컬럼을 하나 만들고 이 컬럼 값에 따라 다른 클래스로 매핑시키는 방법이다.

## Rails 예제 

* 행복할껄 마트는 식자재 전문 도매 마트로 발돋움하려고 애쓰고 있다.
* 그 일환으로 야채만 취급하던 행복할껄 마트는 이번에 소화 잘되는 고기와 양념을 들이려고 한다. 
* 그런데 행복할껄 마트는 통계상 1년에 1번 신규 품목을 들인다. 신규 품목이 없는 해도 있다.
* 신규 품목 추가일을 맡게된 개발자 A씨. 
* 자 그럼 A는 어떻게 해야할까?
* A씨는 신규 품목을 추가할때마다 코드를 수정하기로 정했다. 
* 어쩌다 있는 수정을 위해 애써 관리툴을 만들진 않기로 했다.(!)
* A씨는 이리저리 찾아보다가 Single Table Inheritance 개념을 찾아보곤 적용해보았다
* 아래는 그 적용 예제다.

* 마이그레이션 파일 
```
class AddItem < ActiveRecord::Migration
 def change
    create_table :items do |t|
      t.string :name
      t.string :type
      # 중략
      t.timestamps 
    end
  end
end
```

* 모델 파일
```
class Item < ActiveRecord::Base
  # type 컬럼을 STI의 기준으로 본다.
  self.inheritance_column = :type
end 

class Source < Item
end 

class Meat < Item
end 

```

* 그럼 이제 items 테이블엔 type 컬럼 값이 Source, Meat row가 생기기 시작한다.
* 아래와 같이 조회하면 Source의 인스턴스를 리턴한다.
```
 Item.where(type: 'Source') 
```

## ERB 헬퍼메소드 사용시 주의점

###  STI를 사용하는 모델을 같은 form에서 수정시
* form_for를 사용하면 id가 할당한 모델 클래스 이름을 input id, name의 prefix로 사용한다. 
* 위 예제에선 meat[name] 과 같이 생성한다. 
* 이런 ROR 동작 방식 때문에 form_for 에 STI를 쓴 경우엔 form_for에 as를 꼭 써줘야 입력 파라미터를 못찾는다는 오류가 없다.
* as로 슈퍼 클래스인 item으로 지정하면 모델 클래스에 상관 없이 동일한 포맷으로 form을 전송할 수 있다.
```
<%= form_for(@item, as: :item, url: url, method: method, :html => {:multipart => true, class: "form-horizontal item_form" } ) do |f| %>
```

## STI 모델 subclass가 없어도 오류를 발생 않아야 하는 경우
* 서브 클래스가 삭제 이름 변경의 가능성 있거나 서브 클래스가 사라져도 데이터는 출력해야 하는 경우가 있다.
* 내 경우는 관리툴에서 발생했다. 이 데이터를 관리 해야 하는 경우, 서브 클래스가 없어도 데이터는 출력해야 했다.
* 해결책은 간단했다. 같은 테이블을 바라보는 model만 하나 더 추가한다. 그리고 관리툴에선 이 model을 사용하면 된다.
```
class Model < ActiveRecord:Base
  self.table_name = "items"
end
```
* 이러면 subclass exception의공포 없이 문제를 해결할 수 있다.

## 참고 

* http://samurails.com/tutorial/single-table-inheritance-with-rails-4-part-1/
* http://www.alexreisner.com/code/single-table-inheritance-in-rails
* http://api.rubyonrails.org/classes/ActiveRecord/Inheritance.html
* http://eewang.github.io/blog/2013/03/12/how-and-when-to-use-single-table-inheritance-in-rails/

