使对象类型支持循环垃圾回收
**************************

Python 对循环引用的垃圾检测与回收需要“容器”对象类型的支持，此类型的容
器对象中可能包含其它容器对象。不保存其它对象的引用的类型，或者只保存原
子类型（如数字或字符串）的引用的类型，不需要显式提供垃圾回收的支持。

若要创建一个容器类，类型对象的 "tp_flags" 字段必须包含
"Py_TPFLAGS_HAVE_GC" 并提供一个 "tp_traverse" 处理的实现。如果该类型的
实例是可变的，还需要实现 "tp_clear" 。

Py_TPFLAGS_HAVE_GC

   设置了此标志位的类型的对象必须符合此处记录的规则。为方便起见，下文
   把这些对象称为容器对象。

容器类型的构造函数必须符合两个规则：

1. 必须使用 "PyObject_GC_New()" 或 "PyObject_GC_NewVar()" 为这些对象分
   配内存。

2. 初始化了所有可能包含其他容器的引用的字段后，它必须调用
   "PyObject_GC_Track()" 。

TYPE* PyObject_GC_New(TYPE, PyTypeObject *type)

   类似于 "PyObject_New()" ，适用于设置了 "Py_TPFLAGS_HAVE_GC" 标签的
   容器对象。

TYPE* PyObject_GC_NewVar(TYPE, PyTypeObject *type, Py_ssize_t size)

   类似于 "PyObject_NewVar()" ，适用于设置了 "Py_TPFLAGS_HAVE_GC" 标签
   的容器对象。

TYPE* PyObject_GC_Resize(TYPE, PyVarObject *op, Py_ssize_t newsize)

   为 "PyObject_NewVar()" 所分配对象重新调整大小。 返回调整大小后的对
   象或在失败时返回 "NULL"。 *op* 必须尚未被垃圾回收器追踪。

void PyObject_GC_Track(PyObject *op)

   把对象 *op* 加入到垃圾回收器跟踪的容器对象中。对象在被回收器跟踪时
   必须保持有效的，因为回收器可能在任何时候开始运行。在 "tp_traverse"
   处理前的所有字段变为有效后，必须调用此函数，通常在靠近构造函数末尾
   的位置。

同样的，对象的释放器必须符合两个类似的规则：

1. 在引用其它容器的字段失效前，必须调用 "PyObject_GC_UnTrack()" 。

2. 必须使用 "PyObject_GC_Del()" 释放对象的内存。

void PyObject_GC_Del(void *op)

   释放对象的内存，该对象初始化时由 "PyObject_GC_New()" 或
   "PyObject_GC_NewVar()" 分配内存。

void PyObject_GC_UnTrack(void *op)

   从回收器跟踪的容器对象集合中移除 *op* 对象。 请注意可以在此对象上再
   次调用 "PyObject_GC_Track()" 以将其加回到被跟踪对象集合。 释放器
   ("tp_dealloc" 句柄) 应当在 "tp_traverse" 句柄所使用的任何字段失效之
   前为对象调用此函数。

在 3.8 版更改: "_PyObject_GC_TRACK()" 和 "_PyObject_GC_UNTRACK()" 宏已
从公有 C API 中移除。

"tp_traverse" 处理接收以下类型的函数形参。

int (*visitproc)(PyObject *object, void *arg)

   传给 "tp_traverse" 处理的访问函数的类型。*object* 是容器中需要被遍
   历的一个对象，第三个形参对应于 "tp_traverse" 处理的 *arg* 。Python
   核心使用多个访问者函数实现循环引用的垃圾检测，不需要用户自行实现访
   问者函数。

"tp_traverse" 处理必须是以下类型：

int (*traverseproc)(PyObject *self, visitproc visit, void *arg)

   用于容器对象的遍历函数。 它的实现必须对 *self* 所直接包含的每个对象
   调用 *visit* 函数，*visit* 的形参为所包含对象和传给处理程序的 *arg*
   值。 *visit* 函数调用不可附带 "NULL" 对象作为参数。 如果 *visit* 返
   回非零值，则该值应当被立即返回。

为了简化 "tp_traverse" 处理的实现，Python提供了一个 "Py_VISIT()" 宏。
若要使用这个宏，必须把 "tp_traverse" 的参数命名为 *visit* 和 *arg* 。

void Py_VISIT(PyObject *o)

   如果 *o* 不为 "NULL"，则调用 *visit* 回调函数，附带参数 *o* 和
   *arg*。 如果 *visit* 返回一个非零值，则返回该值。 使用此宏之后，
   "tp_traverse" 处理程序的形式如下:

      static int
      my_traverse(Noddy *self, visitproc visit, void *arg)
      {
          Py_VISIT(self->foo);
          Py_VISIT(self->bar);
          return 0;
      }

"tp_clear" 处理程序必须为 "inquiry" 类型，如果对象不可变则为 "NULL"。

int (*inquiry)(PyObject *self)

   丢弃产生循环引用的引用。不可变对象不需要声明此方法，因为他们不可能
   直接产生循环引用。需要注意的是，对象在调用此方法后必须仍是有效的（
   不能对引用只调用 "Py_DECREF()" 方法）。当垃圾回收器检测到该对象在循
   环引用中时，此方法会被调用。
