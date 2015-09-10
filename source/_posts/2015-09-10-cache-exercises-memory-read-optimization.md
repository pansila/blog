title: cache exercises - Ex. 1 memory read optimization
date: 2015-09-10 13:02:37
tags:
category:
---
Cache�����ִ����������������ж���Ҫ�Ͳ���˵�ˣ���������ⷽ�����ϰ��������ʵ���¡�
ԭ�����Բ����������ͻ�[����](http://blog.jobbole.com/89759/)
��һ���ӽ�cacheһ�δ�memory��ȡ�Ĵ�С�����������cache line��һ�µģ�������ٿ���ͨ��ͳ�Ƶó���
ԭ��ʹ����C#���ӣ��������ѧϰpython��������ģ���¡�
```python
buf_len = 64*1024*1024
buffer = [0] * buf_len

def calc_at_every(step):
	i = 0
	while i < buf_len:
		buffer[i] *= 3
		i += step
		
if __name__ == '__main__':
	from timeit import Timer
	results = []
	for i in range(1, 33):
		#print "test starts for step %d" % i
		t = Timer("calc_at_every(%d)" % i, "from __main__ import calc_at_every")
		#print t.timeit(1)
		results.append(t.timeit(1))
	
	import matplotlib.pyplot as plt
	plt.plot(range(1, 33), results)
	plt.ylabel('runtime per step size')
	plt.xlabel('set size')
	plt.show()
```
��ֱ�ӵ�ģ�¾���list����ϧ�����3��������������ͼ��
![list test](/img/2015-09-10_130439.png)
����Ϊ1ʱ������һ����Ҫ12��֮�࣬��������80ms����Ҹ����£�cache�϶�û�з������á���ʵ������step�ӱ�������ʱ����룬������ָ����������ȫ����cache������
��ʵ��python��list��C/C++�е�array����һ������һ����װ��������Ϊ����array�Ķ�����ռ�õ�sizeԶԶ����Ԫ�ر���֮��
`>>> sys.getsizeof([])`
`64`
������ʵ�Ѿ������Ż�����û��ʹ��`for ... in ...`������pythonic������ʽ����Ϊ����һ������list��ռ��ͬ�����memory������Ҳ��memory read��cache���⡣�ѵ���python�����ܲ��Ծ��Ǹ����ߡ�����

��Ȼlist�����죬�ǻ��ܹ�ֱ�ӽӴ�buffer��bytearray��
```python
from array import array

buf_len = 64*1024*1024
buf = [0] * buf_len
buffer = array('i', buf)

def calc_at_every(step):
	i = 0
	while i < buf_len:
		buffer[i] *= 3
		i += step
		
if __name__ == '__main__':
	from timeit import Timer
	results = []
	for i in range(1, 33):
		#print "test starts for step %d" % i
		t = Timer("calc_at_every(%d)" % i, "from __main__ import calc_at_every")
		#print t.timeit(1)
		results.append(t.timeit(1))
	
	import matplotlib.pyplot as plt
	plt.plot(range(1, 33), results)
	plt.ylabel('runtime per step size')
	plt.xlabel('set size')
	plt.show()
```
����������
![bytearray test](/img/2015-09-10_133710.png)
---
**to be continued**