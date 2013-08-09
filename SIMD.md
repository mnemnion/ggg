#A SIMD implementation of GLL

The Gll algorithm is well suited to parallelization. An architecture which seems promising is OpenCL. The rules may be written as kernels, which run in parallel across the same data. They may be flexibly executed in a task parallel fashion, and may be halted and restarted on the same structures efficiently. 

The reference GGG implementation will be in ECL and C, with the latter written with an eye towards OpenCL implemenation. We can likely handle the situation with the preprocessor, which is a notoriously capable beast. 

Since the `alt` function returns on success, we can pause any other branches where they are, since they immediately write their cursors to a buffer. We can then reuse all those kernels to pursue the next round of success, and if we exit out of a level, wipe all the associated buffers and deallocate the memptrs. If we hit a failure, we instantly restart on the required level. 
