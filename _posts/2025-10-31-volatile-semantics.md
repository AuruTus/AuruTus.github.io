## Why is `volatile` ? The semantic of `volatile`

When we hand code to the compiler, it only sees named memory locations and the need to move data between registers and memory. This means it cannot understand or handle the __mapping__ between the __abstract states__ (e.g., hardware status or concurrency information) expressed by the programming language's symbolic system and the __actual physical memory__.

As a result, a variable or code snippet that is crucial at the abstract level may appear __redundant__ to the compiler. Such code is therefore prone to being altered or removed by the compiler's optimizations. This is precisely why we mark that code as volatile &mdash; to inform the compiler of its special status (, or __side effects__).

Thus, to the compiler, the code itself is in fact __`volatile`__.