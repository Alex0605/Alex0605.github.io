---
title: Android知识(一)--View事件传递
date: 2018-02-24 16:35:09
tags: [Android]
---

### 1、View事件基础知识

　　(1) 所有 Touch 事件都被封装成了 MotionEvent 对象，包括 Touch 的位置、时间、历史记录以及第几个手指(多指触摸)等；

　　(2) 事件类型分为 ACTION_DOWN, ACTION_UP, ACTION_MOVE, ACTION_POINTER_DOWN, ACTION_POINTER_UP, ACTION_CANCEL，每个事件都是以 ACTION_DOWN 开始 ACTION_UP 结束；

　　(3) 对事件的处理包括三类，分别为传递——dispatchTouchEvent()函数、拦截——onInterceptTouchEvent()函数、消费——onTouchEvent()函数和 OnTouchListener；

　　(4)、事件一定是先到达父控件上；

　　(5)、父控件和父类不是一回事，这两个概念初学者很容易混淆。

### 2、View事件模型

​	所谓的事件模型就是：控件→子控件

​	事件模型主要涉及到3个概念：事件的分发、事件的拦截、事件的响应。

#### ​	2.1、事件分发主要分为以下三种情况

​		A、首先会先调用自身的onInterceptTouchEvent方法，调用此方法的目的是为了，先让自己这个控件判断下是否需要把此事件拦截下来，如果拦截下来，那么就代表自己这个控件需要处理这个事件，所以此时会调用自身onTouchEvent来对这个事件进行响应。

​		B、如果不拦截下来，那么才会有后续的事件向下传递的流程。将这个事件传递给子控件。现在子控件接收到了这个事件，上文提过，一个事件到达一个View或者ViewGroup，就会最先调用这个控件的dispatchTouchEvent，所以此时，事件到达子控件的dispatchTouchEvent方法，如果这个控件仍然是一个ViewGroup的类型，那么事件继续分发的逻辑依然遵循A流程的逻辑。

​		C、如果这个子控件只是一个View，而不是ViewGroup，那么此时，事件分发的逻辑略有不同。由于View没有onInterceptTouchEvent的方法，所以当一个事件到达这个View的dispatchTouchEvent的时候，dispatchTouchEvent就调用不到onInterceptTouchEvent，它会直接调用onTouchEvent的方法，直接让这个View来响应此事件。

#### ​	2.2、事件响应

![event_respose](Android知识-View事件传递/event_respose.jpg)

​        如上图所示，如果ViewGroupB拦截了事件，那么此时事件就会由ViewGroupB来响应，调用ViewGroupB中的onTouchEvent，此时ViewGroupB中的onTouchEvent的返回值有两种可能，一种是true，一种是false，如果返回true，则代表ViewGroupB消费了此事件，事件此时终止。如果返回的值是false，那么此时这个事件会回传给父控件，调用到父控件的onTouchEvent方法，由父控件来进行响应，那父控件的onTouchEvent也是同样的逻辑。要么消费事件，要么回传给父控件的父控件。
　　此时，就可以得出我们通常所说的两个方向：
　　(1)、事件传递的方向：父控件→子控件
　　(2)、事件响应的方向：子控件→父控件	

​	当然，仅仅是这个结论是无法满足我们实际开发的需要，我们需要更细致的分析。这里有一个细节上的问题需要注意，就是事件分为Down事件、Move事件、Up事件，任何一种事件都遵循事件传递和响应的逻辑原则，很多开发者常常会认为Down-Move-Up连在一起才是一个事件的产生，这种想法是不对的。
事件的起点是由Down事件开始的，然后产生一系列的Move事件，最后通常以Up事件结束。当Down事件产生的时候，会由父控件传递给子控件，Move事件也由父控件传递给子控件，Up事件也由父控件传递给子控件。它们都遵循同样的传递事件的逻辑流程。不过Down事件最终响应的结果，会影响到后续事件的执行。这句话是什么意思呢？

#### ​	2.3 、事件拦截

​	如果Down事件传递到了子View上，但是子View的onTouchEvent对于这个Down事件的处理是return了一个false，这样的结果就是会造成父View的onTouchEvent的调用，同时还有另外一个后果，那就是后续的Move事件、Up事件就都传递不到子View上。所以，如果一个View要处理滑动事件，也就是Move事件的话，那么它一定不能在onTouchEvent中，对Down事件return false。

　　如果Down事件到了父View上，父View需要调用自身的onInterceptTouchEvent判断是否对这个Down事件进行拦截，如果拦截，return了true，那么这个事件就会到父View的onTouchEvent中进行响应。如果此时父View的onTouchEvent也返回了true，那么代表这个父View响应了Down事件。不过这里有一点不太一样的地方是，事件传递到父View的onTouchEvent方法是因为自身的onInterceptTouchEvent方法判断拦截导致的，而不是由子View回传回来的，在这种情况下，当Move事件、Up事件传递到父View的时候，它当然不会传递给子View，并且，也不再调用自身的onInterceptTouchEvent方法。

#### 	2.4 、事件冲突的解决

　　理解事件传递的基本逻辑，对于工作过程中解决滑动事件冲突非常有帮助。比如我们此时有一个父控件ViewPager，这个ViewPager其中一个Item是ScrollView，此时会发生什么问题呢？当ViewPager滑动到ScrollView这个条目的时候，再左右滑动，发现ViewPager再也左右滑动不了了。这是为什么呢？我们结合图6一起来分析一下。


　　(1)、我们都知道ViewPager是能够横向滑动的控件，而ScrollView是纵向滑动的控件，当Down事件产生的时候，此时会由ViewPager传递给ScrollView，ViewPager没有对Down事件拦截，ScrollView也不会对这个Down事件进行拦截，所以事件就会传递给ScrollView的孩子，也就是类似于图6中的子View，子View如果没有对Down事件响应，那么最后会到ScrollView中的onTouchEvent，而ScrollView的onTouchEvent对于这个Down事件返回了true，代表ScrollView消费了这个Down事件。


　　(2)、接下来开始滑动手指，产生一系列的Move事件。Move事件也是由ViewPager传递给ScrollView。由于Down事件是被ScrollView的onTouchEvent中消费的，所以Move事件就不会传递给ScrollView的子控件了。一系列的Move事件也是在ScrollView的onTouchEvent中被执行。


　　(3)、最后的Up事件也是由ScrollView中的onTouchEvent消费。


　　从上述1至3的步骤中，我们看出来无论是Down事件、Move事件还是Up事件，最后全部都是被ScrollView所消费。从头到尾ViewPager的onTouchEvent都没有得到执行。而ViewPager之所以能够左右滑动，正是因为ViewPager的onTouchEvent里面的代码逻辑产生的效果。ViewPager的onTouchEvent没有执行，这个ViewPager当然就不能够左右滑动了。所以解决上述问题，就是在于如何让ViewPager中的onTouchEvent方法执行。
我们可以自定义一个MyViewPager继承ViewPager，重写onInterceptTouchEvent方法，如果我们在onInterceptTouchEvent方法中直接野蛮地return一个true，此时就代表无论是Down事件、Move事件，还是Up事件，全部都拦截下来了，拦截在MyViewPager中，我们可以认为是图6中的ViewGroupB，既然拦截下来了所有事件，那么所有事件就会传递到MyViewPager的onTouchEvent，所以此时，这个MyViewPager一定可以左右滑动。

　　但是，由此会引发另外一个问题，就是这个ScrollView不能上下滑动了。这又是为什么呢？因为ScrollView能够上下滑动的代码逻辑在ScrollView中的onTouchEvent方法内，而此时事件又全部被MyViewPager拦截了下来，ScrollView完全得不到事件，onTouchEvent方法得不到执行，自然不能上下滑动。所以我们需要修改MyViewPager中的onInterceptTouchEvent的逻辑。


　　ViewPager只对左右滑动感兴趣，而ScrollView对上下滑动这个动作感兴趣，所以我们只需要在MyViewPager的onInterceptTouchEvent中，根据多个Move事件，判断是左右滑动还是上下滑动，如果是左右滑动，return true将事件拦截下来，如果是上下滑动，return false将事件传递给ScrollView，这样就能解决问题了。
所以，对于Down事件，我们一般都不进行拦截，判断是否拦截得根据一些列的Move事件才能得出具体的条件是否成立。

#### 	2.5、Cancel事件的产生：

　　刚才我们说了事件一般有三个，Down、Move、Up，这三个事件比较好理解。其实还有一种事件就是Cancel事件。它代表什么含义呢？
还是回到图6，如果一个Down事件产生了，这个Down事件从ViewGroupA传递到ViewGroupB，最终到达子View，被子View的onTouchEvent消费，return了true，那么此时Down事件就终止了。接下来后续的Move事件也会从ViewGroupA传递给ViewGroupB，也就是说ViewGroupA和ViewGroupB会比子View更先拿到Move事件，那既然ViewGroupA和ViewGroupB比子View更先拿到Move事件，那么他们当中的任何一个都有可能在某一个Move事件中，把这个Move事件给拦截下来，一旦Move事件被拦截下来了，子View肯定就拿不到这个Move事件了，不过，此时子View会产生一个新的事件，就是Cancel事件。


　　所以一个正常的事件序列是 Down→Move→Up,这样才被认为是一个正常的事件序列。如果一个View响应的Down事件，可是却被没有正常结尾，Move事件或者Up事件被拦截了，此时非正常结尾的情况就会给子View产生一个新的事件Cancel。

#### 	2.7、子控件可以影响父控件是否拦截的行为

　　子控件是可以干预父控件是否拦截事件的结果。通过在子View中dispatchTouchEvent中增加一行代码即可。getParent().requestDisallowInterceptTouchEvent(true);这行代码就可以请求父控件不要拦截事件。


　　很多人可能不太明白这句话的意思，既然事件一定是先到达父控件，然后才到达子View，那也就是getParent().requestDisallowInterceptTouchEvent(true);这句话是在父控件是否拦截判断结束之后才调用，怎么能改变父控件是否拦截的结果呢，这里存在一个执行先后顺序的疑惑。
　　

　　其实是这样的，getParent().requestDisallowInterceptTouchEvent(true);达到的效果不是修改父控件对本次事件是否拦截的结果，而影响的是后续事件。比如子View在Down事件中调用了getParent().requestDisallowInterceptTouchEvent(true);这行代码，那么在后续Move事件、Up事件产生到达父控件的时候，父控件就不会再拦截了。所以getParent().requestDisallowInterceptTouchEvent(true);只会影响Move事件和Up事件，影响不到Down事件。