---
layout: post
title: [how2heap] fastbin_dup

date: 2019-03-01
tags: [heap]

---

fastbin_dup



fastbin_dup.c
```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
        printf("이 파일은 fastbin에서의 간단한 double-free 공격을 보여줍니다.\n");

        printf("먼저 3개의 버퍼를 할당합니다.\n");
        int *a = malloc(8);
        int *b = malloc(8);
        int *c = malloc(8);

        printf("1st malloc(8): %p\n", a);
        printf("2nd malloc(8): %p\n", b);
        printf("3rd malloc(8): %p\n", c);

        printf("그 다음 첫 번째를 free합니다...\n");
        free(a);

        printf("만약 우리가 %p를 다시 free한다면 , 크래시가 날 것입니다. 왜냐하면 %p는 free리스트의 가장 위에 있기 때문입니다.\n", a, a);
        // free(a);

        printf("그러므로, 우리는 %p를 대신 free할 것입니다.\n", b);
        free(b);

        printf("이제 우리는 다시 %p를 free할 수 있습니다. 왜냐하면 이 이후로 더 이상 free리스트가 맨 위에 있지 않기 때문입니다.\n", a);
        free(a);

        printf("이제 free리스트는 [ %p, %p, %p ]를 가지고 있습니다. 만약 우리가 malloc을 다시 3번 한다면, %p를 두 번 할당받을 수 있습니다.\n", a, b, a, a);
        printf("1st malloc(8): %p\n", malloc(8));
        printf("2nd malloc(8): %p\n", malloc(8));
        printf("3rd malloc(8): %p\n", malloc(8));
}
```

설명 추가하고 싶은데 시간 날 때 추가하겠습니당