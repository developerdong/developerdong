# AX程序放到Jupiter上执行的整个流程

用AX编写程序，然后由omsp_jupiter_proxy传到Jupiter，
然后用omspAdapter编译成plcapp可以识别的代码后放到共享内存，
再由plcapp读取执行。
plcapp通过DDS从rsc的time_service获取cyclic event，
每一个cyclic event都会触发一个子进程，
子进程会向rsc的task_service注册task，
然后如果task被调度，则执行user program，
如果没调度则这个cyclic event不执行用户程序。
程序执行时从DDM+DDA（底层是RIB）读取输入数据，
以及向它们写入输出数据。
