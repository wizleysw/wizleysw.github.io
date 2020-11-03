---
published: true
layout: single
title : "ViewModel에 대한 이야기"
category : android
comments: false
author_profile : true
tag : 
  - Android
toc : true
---

## OverView

안드로이드의 LifeCycle을 공부하다보면 드는 여러 의문 가운데 하나가 로테이션과 관련된 것이다. 예를 들어 사용자가 화면을 가로에서 세로로 전환하게 된 경우에 발생하는 상황에 관한 것인데 LifeCycle에 따르면 onDestroy가 호출되고 새로 호출되는 Activity에서 layout이 inflate 되게 된다.

### OnSavedInstanceState ??

여기서 하나의 의문이 들었다. 만약 사용자가 여러 정보를 Activity에 가지고 있는 상황에서 화면에 대한 전환이 일어나게 되면 그 데이터들이 어떻게 유지될 수 있느냐 이다. onSavedInstance가 존재한다. 그리고 이는 UI와 관련된 여러 값들을 유지하기 위한 작은 공간을 의미한다. 그렇기 때문에 다음의 두 상황에서 주로 사용된다.

첫 번째로는 메모리가 부족하여 백그라운에 onStop 상태로 존재하는 App의 데이터를 보호하기 위함이며 두 번째로는 Configuration이 변경되었을 경우이다. EditText와 같이 UI와 연관된 여러 값들이 저절로 저장됨을 의미한다.

그렇다면 왜 OnSavedInstanceState만을 사용할 수는 없는걸까? 바로 앞에서 말했듯 적은 양의 데이터만을 보호할 수 있기 때문이다. Bitmap과 같이 큰 용량을 차지하는 값들에 대한 보호가 불가능하다. 그렇기 때문에 보관할 수 있는 데이터들도 serialize가 되는 소량의 데이터로 한정된다. 만약 여러번 데이터 복구작업이 수행되면 그만큼 과부하가 됨을 의미하기도 한다. 

그래서 사실상 Android system에 의해서 Destroyed 된 값들에 대한 복구의 목적으로 사용된다. 

ViewModel은 Application Process의 메모리상에 위치하여 configuration 즉, 설정에 대한 변경상황에서 살아남게 되는 공간으로 용량이 큰 데이터들을 저장하기에 용이하다. 

### LifeCycle of ViewModel

![ViewModel lifecycle](https://developer.android.com/images/topic/libraries/architecture/viewmodel-lifecycle.png)

ViewModel의 경우 Finish()가 호출되기 전까지 계속 유지가 되는 객체이기 때문에 화면 전환등으로 인하여 onDestroy가 호출되어 onCreate 부분으로 돌아가더라도 데이터가 유지된다. 근데 여기서 또 하나의 의문이 든다. 

onDestroy는 Activity가 가로에서 세로로 Rotation이 되는 상황에도 호출이 되는데 이를 어떻게 구별할까? 정답은 간단했다. isFinishing()이 이를 판별해주기 때문이다. Finish는 명확하게 해당 Activity가 더이상 사용되지 않음을 말해주는데 isFinishing이 True가 아닌 경우에는 System에 의해 Activity가 onDestroy되거나 화면이 전환되는 것과 같은 상황을 판별할 때 사용될 수 있다. 

```java
protected void onDestroy() {
    super.onDestroy();
    if(isFinishing()){
        Log.e(TAG, "isFinishing");
    } else {
        Log.e(TAG, "Rotating orientation");
    }
}
```

로테이션이 일어나는 순간에 onDestroy 부분에 isFinishing 여부를 로깅을 해보면 아래와 같이 상황을 구별하는 것을 확인할 수 있다.    

```
2020-06-17 15:11:31.843 26179-26179/wizley.android.playground E/ViewModelCounterObserveActivity: Rotating orientation
2020-06-17 15:11:41.076 26179-26179/wizley.android.playground E/ViewModelCounterObserveActivity: Rotating orientation
```

### ViewModel Example

그렇다면 이번에는 실습을 해볼 시간이다. 여기서 시간을 많이 소요하였는데 기존에 사용하던 ViewModelProviders가 deprecated되었고, Google Developer Docs의 코드가 해당 버전으로부터 또 변경되어서 해당 코드가 정상적으로 작동하지 않았기 때문이다. 여러 삽질 끝에 성공하여 다음과 같이 글을 남길 수 있게 되었다.

```java
package wizley.android.playground.viewmodel.counter;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

public class CounterViewModel extends ViewModel {
    private MutableLiveData<Integer> counter = new MutableLiveData<>();

    public CounterViewModel(){
        counter.setValue(0);
    }

    public void increase(){
        counter.setValue(counter.getValue()+1);
    }

    public void decrease(){
        if(counter.getValue() >= 1) {
            counter.setValue(counter.getValue() - 1);
        }
    }

    public MutableLiveData<Integer> getCounter(){
        return this.counter;
    }

    public Integer getValue(){
        return this.counter.getValue();
    }

    public void setValue(Integer val){
        counter.setValue(val);
    }
}
```

CustomViewModel을 적용하는 법은 다음과 같다. 여기서 MutableLiveData를 사용하였는데 Integer형으로 선언하더라도 사용이 가능하다. 이렇게 선언된 ViewModel이 Activity에서 관리되게 되면 finish()가 호출되기 전까지 counter라는 변수에 대한 값이 유지된다. 

MutableLiveData는 Data가 변경되었는지를 확인할 때 사용되는데 옵저버를 추가하여 값에 대한 변경이 일어나면 특정 행위를 하도록 하는 것이 가능하다.

```java
public class ViewModelCounterActivity extends AppCompatActivity implements View.OnClickListener {
    private static final String TAG = "ViewModelCounterActivity";

    private FloatingActionButton fabPlus;
    private FloatingActionButton fabMinus;
    private TextView textView;

    CounterViewModel model;

    private int counter = 0;

    @SuppressLint("SetTextI18n")
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_viewmodel_counter);

        model = new ViewModelProvider(this,
                ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication())).get(CounterViewModel.class);

        fabPlus = (FloatingActionButton) findViewById(R.id.fabPlus);
        fabMinus = (FloatingActionButton) findViewById(R.id.fabMinus);
        textView = (TextView) findViewById(R.id.textView);

        textView.setText(model.getValue() + "");

        fabPlus.setOnClickListener(this);
        fabMinus.setOnClickListener(this);
    }

    @SuppressLint("SetTextI18n")
    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.fabPlus:
                model.increase();
                textView.setText(model.getValue() + "");
                break;
            case R.id.fabMinus:
                if(counter >= 1) {
                    model.decrease();
                    textView.setText(model.getValue() + "");
                }
                break;
        }
    }
}
``` 

onCreate 부분에서 ViewModelProvider를 만드는 부분에서 CounterViewModel이 연결된다. 해당 부분이 Docs와 다른 부분인데 2nd 파라미터로 ModelFactory를 가지고 있어야 된다. model이 추가되고 나면 counter의 값은 finish가 호출되는 순간까지 유지되기 때문에 onClickListener를 통해 increase, decrease된 값이 화면이 rotate되더라도 살아남게 된다.

```java
public class ViewModelCounterObserveActivity extends AppCompatActivity implements View.OnClickListener {
    private static final String TAG = "ViewModelCounterObserveActivity";

    private FloatingActionButton fabPlus;
    private FloatingActionButton fabMinus;
    private TextView textView;

    CounterViewModel model;

    @SuppressLint("SetTextI18n")
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_viewmodel_counter);

        model = new ViewModelProvider(this,
                ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication())).get(CounterViewModel.class);

        fabPlus = (FloatingActionButton) findViewById(R.id.fabPlus);
        fabMinus = (FloatingActionButton) findViewById(R.id.fabMinus);
        textView = (TextView) findViewById(R.id.textView);

        textView.setText(model.getValue() + "");

        fabPlus.setOnClickListener(this);
        fabMinus.setOnClickListener(this);

        final Observer<Integer> counterObserver = new Observer<Integer>() {
            @Override
            public void onChanged(Integer integer) {
                textView.setText(integer + "");
            }
        };

        model.getCounter().observe(this, counterObserver);
    }

    @SuppressLint("LongLogTag")
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(isFinishing()){
            Log.e(TAG, "isFinishing");
        } else {
            Log.e(TAG, "Rotating orientation");
        }
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.fabPlus:
                model.increase();
                break;
            case R.id.fabMinus:
                model.decrease();
                break;
        }
    }
}
```

이번에는 LiveMutableData을 활용하기 위해서 observer를 추가하였다. counterObserver는 getCounter와 연결되어있는지 counter의 값이 변경되게 되면 onChanged 부분이 호출되어 textView의 숫자 값이 변경되게 된다. 그 덕에 onClick에서 따로 setText를 호출할 필요없이 값이 바뀌는 순간마다 저절로 호출이 되어 값이 최신화 된다. 

여기서 AppCompatActivity를 사용하지 않고 Activity를 사용한다면 LifeCycleOwner 인터페이스를 추가하여 observe의 첫 번째 인자로 owner를 넣어주어야 되기 때문에 내부적으로 LifeCycleOwner를 가지고 잇는 AppCompatActivity를 사용하기를 추천하는 바이다. 

