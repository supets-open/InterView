
[Fragment在不同情况下的生命周期](https://blog.csdn.net/jokeeeeee/article/details/46004931)


1、Fragment在Activity中replace

新替换的Fragment：onAttach > onCreate > onCreateView > onViewCreated > onActivityCreated > onStart > onResume

被替换的Fragment：onPause > onStop > onDestroyView > onDestroy > onDetach


2、Fragment在Activity中replace，并addToBackStack

新替换的Fragment（没有在BackStack中）：onAttach > onCreate > onCreateView > onViewCreated > onActivityCreated > onStart > onResume

新替换的Fragment（已经在BackStack中）：onCreateView > onViewCreated > onActivityCreated > onStart > onResume

被替换的Fragment：onPause > onStop > onDestroyView


3、Fragment在ViewPager中切换

这里有很多种情况，我们先将切换前的的Fragment称为PreviousFragment，简称PF；切换后的Fragment称为NextFragment，简称NF；其他Fragment称为OtherFragment，简称OF。

（在ViewPager中setUserVisibleHint能反映出Fragment是否被切换到后台或前台，所以在这里也当作生命周期）


A、如果相关的Fragment没有被加载过：

NF： setUserVisibleHint(false) > onAttach > onCreate > setUserVisibleHint(true) > onCreateView > onViewCreated > onActivityCreated > onStart > onResume

OF跟NF相邻： setUserVisibleHint(false) > onAttach > onCreate > onCreateView > onViewCreated > onActivityCreated > onStart > onResume


B、如果相关的Fragment已经被加载过：

NF跟PF相邻  ：setUserVisibleHint(true)

NF跟PF不相邻：setUserVisibleHint(true) > onCreateView > onViewCreated > onActivityCreated > onStart > onResume

PF跟NF相邻  ：setUserVisibleHint(false)

PF跟NF不相邻：setUserVisibleHint(false) > onPause > onStop > onDestroyView

OF跟PF相邻：onPause > onStop > onDestroyView

OF跟NF相邻：onCreateView > onViewCreated > onActivityCreated > onStart > onResume

OF夹在PF和NF中间：不调用任何生命周期方法


C、如果重写了FragmentPagerAdapter的destroyItem方法，并且相关Fragment已经加载过：

则相互切换时只会调用setUserVisibleHint


4、Fragment进入了运行状态：

Fragment在上述的各种情况下进入了onResume后，则进入了运行状态，以下4个生命周期方法将跟随所属的Activity一起被调用：

onPause > onStop > onStart > onResume


5、关于Fragment的onActivityResult方法：

使用Fragment的startActivity方法时，FragmentActivity的onActivityResult方法会回调相应的Fragment的onActivityResult方法，所以在重写FragmentActivity的onActivityResult方法时，注意调super.onActivityResult