### Synchronized修饰的方法在抛出异常时,会释放锁吗?

synchronized在发生异常时，会自动释放掉锁，故不会发生死锁现(此时的死锁一般是代码逻辑引起的)；而Lock必须在finally中主动unlock锁，否则就会出现死锁。