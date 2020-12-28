# AK-2 Lab7 
## Виконала: Головаш Анастасія 
## Група: ІВ-82

## Лістинг:

### hello1.h
```
#include <linux/types.h>
int hello(uint n);
```

### hello71.c
```
// SPDX-License-Identifier: GPL-2-Clause
#include <linux/init.h>
#include <linux/module.h>
#include <linux/printk.h>
#include <linux/types.h>
#include <linux/slab.h>
#include <linux/ktime.h>
#include <hello1.h>

MODULE_LICENSE("Dual BSD/GPL");
MODULE_AUTHOR("Holovash Anastasia IV-82\n");
MODULE_DESCRIPTION("AK-2 Lab6 hello1\n");

struct timeit_list {
	struct list_head node;
	ktime_t before;
	ktime_t after;
};

static struct list_head head_node = LIST_HEAD_INIT(head_node);


int hello(uint n)
{
	struct timeit_list *list, *tmp;
	uint i;

	BUG_ON(n > 10);

	if (n <= 0) {
		pr_err("ERROR! n < 0\n");
		return -EINVAL;
	} else if (n == 0) {
		pr_warn("WARNING! n = 0\n");
	} else if (n >= 5 && n <= 10) {
		pr_warn("WARNING! 5 <= n <= 10\n");
	}

	for (i = 0; i < n; i++) {
		list = kmalloc(sizeof(struct timeit_list), GFP_KERNEL);
		if (i == 7)
			list = NULL;
		if (ZERO_OR_NULL_PTR(list))
			goto clean_up;

		list->before = ktime_get();
		pr_info("Hello, world!\n");
		list->after = ktime_get();
		list_add_tail(&list->node, &head_node);
	}
	return 0;

clean_up:
	list_for_each_entry_safe(list, tmp, &head_node, node) {
		list_del(&list->node);
		kfree(list);
	}
	pr_err("ERROR! Memory is out\n");
	return -ENOMEM;
}
EXPORT_SYMBOL(hello);


static int __init init_hello(void)
{
	pr_info("hello1 init\n");
	return 0;
}


static void __exit exit_hello(void)
{
	struct timeit_list *list, *tmp;

	list_for_each_entry_safe(list, tmp, &head_node, node) {
		pr_info("Time: %lld", list->after - list->before);
		list_del(&list->node);
		kfree(list);
	}

	pr_info("hello1 exit\n");
}


module_init(init_hello);
module_exit(exit_hello);
```

### hello72.c
```
// SPDX-License-Identifier: GPL-2-Clause
#include <linux/init.h>
#include <linux/module.h>
#include <linux/printk.h>
#include <linux/types.h>
#include <linux/slab.h>
#include <linux/ktime.h>
#include <hello1.h>

MODULE_LICENSE("Dual BSD/GPL");
MODULE_DESCRIPTION("AK-2 Lab6 hello1\n");
MODULE_AUTHOR("Holovash Anastasia IV-82\n");

static uint n = 1;

module_param(n, uint, 0);
MODULE_PARM_DESC(n, "How many hellos to print\n");

static int __init init_hello(void)
{
	pr_info("hello2 init\n");
	hello(n);
	return 0;
}

static void __exit exit_hello(void)
{
	pr_info("hello2 exit\n");
}

module_init(init_hello);
module_exit(exit_hello);
```
### Makefile
```

ccflags-y := -I$(PWD)/inc
ifneq ($(KERNELRELEASE),)
# kbuild part of makefile
obj-m := hello71.o hello72.o
ccflags-y += -g -DDEBUG
else
# normal makefile
KDIR ?= /lib/modules/`uname -r`/build
default:
	$(MAKE) -C $(KDIR) M=$$PWD
	cp hello71.ko hello71.ko.unstripped
	cp hello72.ko hello72.ko.unstripped
	$(CROSS_COMPILE)strip -g hello71.ko
	$(CROSS_COMPILE)strip -g hello72.ko
clean:
	$(MAKE) -C $(KDIR) M=$$PWD clean
%.s %.i: %.c
	$(MAKE) -C $(KDIR) M=$$PWD $@
endif
```

## Результат роботи програми:
![image1](image1.png)
![image2](image2.png)

## hello71.ko.unstripped
```
hello71.ko.unstripped:     file format elf32-littlearm


Disassembly of section .text:

00000000 <hello>:
int hello(uint n)
{
	struct timeit_list *list, *tmp;
	uint i;

	BUG_ON(n > 10);
   0:	e350000a 	cmp	r0, #10
{
   4:	e92d47f0 	push	{r4, r5, r6, r7, r8, r9, sl, lr}
	BUG_ON(n > 10);
   8:	8a000048 	bhi	130 <hello+0x130>

	if (n <= 0) {
   c:	e3500000 	cmp	r0, #0
  10:	e1a07000 	mov	r7, r0
  14:	0a000046 	beq	134 <hello+0x134>
		pr_err("ERROR! n < 0\n");
		return -EINVAL;
	} else if (n == 0) {
		pr_warn("WARNING! n = 0\n");
	} else if (n >= 5 && n <= 10) {
  18:	e2403005 	sub	r3, r0, #5
  1c:	e3530005 	cmp	r3, #5
  20:	8a000002 	bhi	30 <hello+0x30>
		pr_warn("WARNING! 5 <= n <= 10\n");
  24:	e3000000 	movw	r0, #0
  28:	e3400000 	movt	r0, #0
  2c:	ebfffffe 	bl	0 <printk>
			unsigned int index = kmalloc_index(size);

			if (!index)
				return ZERO_SIZE_PTR;

			return kmem_cache_alloc_trace(kmalloc_caches[index],
  30:	e3008000 	movw	r8, #0
  34:	e3408000 	movt	r8, #0
  38:	e3a010c0 	mov	r1, #192	; 0xc0
  3c:	e3a02018 	mov	r2, #24
  40:	e5980018 	ldr	r0, [r8, #24]
  44:	e3401060 	movt	r1, #96	; 0x60
  48:	ebfffffe 	bl	0 <kmem_cache_alloc_trace>
  4c:	e3005000 	movw	r5, #0
			list = NULL;
		if (ZERO_OR_NULL_PTR(list))
			goto clean_up;

		list->before = ktime_get();
		pr_info("Hello, world!\n");
  50:	e3009000 	movw	r9, #0
  54:	e3a0a0c0 	mov	sl, #192	; 0xc0
  58:	e2677008 	rsb	r7, r7, #8
  5c:	e3405000 	movt	r5, #0
  60:	e3409000 	movt	r9, #0
  64:	e340a060 	movt	sl, #96	; 0x60
  68:	e3a06007 	mov	r6, #7
  6c:	e1a04000 	mov	r4, r0
		if (ZERO_OR_NULL_PTR(list))
  70:	e3540010 	cmp	r4, #16
  74:	9a000013 	bls	c8 <hello+0xc8>
		list->before = ktime_get();
  78:	ebfffffe 	bl	0 <ktime_get>
  7c:	e1c400f8 	strd	r0, [r4, #8]
		pr_info("Hello, world!\n");
  80:	e1a00009 	mov	r0, r9
  84:	ebfffffe 	bl	0 <printk>
		list->after = ktime_get();
  88:	ebfffffe 	bl	0 <ktime_get>
 * Insert a new entry before the specified head.
 * This is useful for implementing queues.
 */
static inline void list_add_tail(struct list_head *new, struct list_head *head)
{
	__list_add(new, head->prev, head);
  8c:	e5953004 	ldr	r3, [r5, #4]
	for (i = 0; i < n; i++) {
  90:	e1560007 	cmp	r6, r7
	new->next = next;
  94:	e5845000 	str	r5, [r4]
	next->prev = new;
  98:	e5854004 	str	r4, [r5, #4]
	new->prev = prev;
  9c:	e5843004 	str	r3, [r4, #4]
		list->after = ktime_get();
  a0:	e1c401f0 	strd	r0, [r4, #16]
static __always_inline void __write_once_size(volatile void *p, void *res, int size)
{
	switch (size) {
	case 1: *(volatile __u8 *)p = *(__u8 *)res; break;
	case 2: *(volatile __u16 *)p = *(__u16 *)res; break;
	case 4: *(volatile __u32 *)p = *(__u32 *)res; break;
  a4:	e5834000 	str	r4, [r3]
	for (i = 0; i < n; i++) {
  a8:	0a00001e 	beq	128 <hello+0x128>
  ac:	e3a02018 	mov	r2, #24
  b0:	e1a0100a 	mov	r1, sl
  b4:	e5980018 	ldr	r0, [r8, #24]
  b8:	ebfffffe 	bl	0 <kmem_cache_alloc_trace>
		if (i == 7)
  bc:	e2566001 	subs	r6, r6, #1
  c0:	e1a04000 	mov	r4, r0
  c4:	1affffe9 	bne	70 <hello+0x70>
		list_add_tail(&list->node, &head_node);
	}
	return 0;

clean_up:
	list_for_each_entry_safe(list, tmp, &head_node, node) {
  c8:	e5953000 	ldr	r3, [r5]
  cc:	e1530005 	cmp	r3, r5
  d0:	e5934000 	ldr	r4, [r3]
}

static inline void list_del(struct list_head *entry)
{
	__list_del_entry(entry);
	entry->next = LIST_POISON1;
  d4:	13a08c01 	movne	r8, #256	; 0x100
	entry->prev = LIST_POISON2;
  d8:	13a07c02 	movne	r7, #512	; 0x200
  dc:	11a06004 	movne	r6, r4
  e0:	0a00000b 	beq	114 <hello+0x114>
	__list_del(entry->prev, entry->next);
  e4:	e5932004 	ldr	r2, [r3, #4]
		list_del(&list->node);
		kfree(list);
  e8:	e1a00003 	mov	r0, r3
	next->prev = prev;
  ec:	e5842004 	str	r2, [r4, #4]
  f0:	e5824000 	str	r4, [r2]
	entry->next = LIST_POISON1;
  f4:	e5838000 	str	r8, [r3]
	entry->prev = LIST_POISON2;
  f8:	e5837004 	str	r7, [r3, #4]
  fc:	ebfffffe 	bl	0 <kfree>
	list_for_each_entry_safe(list, tmp, &head_node, node) {
 100:	e5944000 	ldr	r4, [r4]
 104:	e1560005 	cmp	r6, r5
 108:	e1a03006 	mov	r3, r6
 10c:	e1a06004 	mov	r6, r4
 110:	1afffff3 	bne	e4 <hello+0xe4>
	}
	pr_err("ERROR! Memory is out\n");
 114:	e3000000 	movw	r0, #0
 118:	e3400000 	movt	r0, #0
 11c:	ebfffffe 	bl	0 <printk>
	return -ENOMEM;
 120:	e3e0000b 	mvn	r0, #11
 124:	e8bd87f0 	pop	{r4, r5, r6, r7, r8, r9, sl, pc}
	return 0;
 128:	e3a00000 	mov	r0, #0
}
 12c:	e8bd87f0 	pop	{r4, r5, r6, r7, r8, r9, sl, pc}
	BUG_ON(n > 10);
 130:	e7f001f2 	.word	0xe7f001f2
		pr_err("ERROR! n < 0\n");
 134:	e3000000 	movw	r0, #0
 138:	e3400000 	movt	r0, #0
 13c:	ebfffffe 	bl	0 <printk>
		return -EINVAL;
 140:	e3e00015 	mvn	r0, #21
 144:	e8bd87f0 	pop	{r4, r5, r6, r7, r8, r9, sl, pc}

Disassembly of section .init.text:

00000000 <init_module>:
EXPORT_SYMBOL(hello);


static int __init init_hello(void)
{
   0:	e92d4010 	push	{r4, lr}
	pr_info("hello1 init\n");
   4:	e3000000 	movw	r0, #0
   8:	e3400000 	movt	r0, #0
   c:	ebfffffe 	bl	0 <printk>
	return 0;
}
  10:	e3a00000 	mov	r0, #0
  14:	e8bd8010 	pop	{r4, pc}

Disassembly of section .exit.text:

00000000 <cleanup_module>:

static void __exit exit_hello(void)
{
	struct timeit_list *list, *tmp;

	list_for_each_entry_safe(list, tmp, &head_node, node) {
   0:	e3003000 	movw	r3, #0
   4:	e3403000 	movt	r3, #0
{
   8:	e92d47f0 	push	{r4, r5, r6, r7, r8, r9, sl, lr}
		pr_info("Time: %lld", list->after - list->before);
   c:	e3007000 	movw	r7, #0
	list_for_each_entry_safe(list, tmp, &head_node, node) {
  10:	e5934000 	ldr	r4, [r3]
		pr_info("Time: %lld", list->after - list->before);
  14:	e3407000 	movt	r7, #0
	entry->next = LIST_POISON1;
  18:	e3a08c01 	mov	r8, #256	; 0x100
  1c:	e1a05003 	mov	r5, r3
	entry->prev = LIST_POISON2;
  20:	e3a09c02 	mov	r9, #512	; 0x200
	list_for_each_entry_safe(list, tmp, &head_node, node) {
  24:	e5946000 	ldr	r6, [r4]
  28:	e1540005 	cmp	r4, r5
  2c:	0a000010 	beq	74 <cleanup_module+0x74>
		pr_info("Time: %lld", list->after - list->before);
  30:	e5941010 	ldr	r1, [r4, #16]
  34:	e1a00007 	mov	r0, r7
  38:	e5942008 	ldr	r2, [r4, #8]
  3c:	e594c014 	ldr	ip, [r4, #20]
  40:	e594300c 	ldr	r3, [r4, #12]
  44:	e0512002 	subs	r2, r1, r2
  48:	e0cc3003 	sbc	r3, ip, r3
  4c:	ebfffffe 	bl	0 <printk>
	__list_del(entry->prev, entry->next);
  50:	e1c420d0 	ldrd	r2, [r4]
		list_del(&list->node);
		kfree(list);
  54:	e1a00004 	mov	r0, r4
	next->prev = prev;
  58:	e5823004 	str	r3, [r2, #4]
  5c:	e5832000 	str	r2, [r3]
	entry->prev = LIST_POISON2;
  60:	e1c480f0 	strd	r8, [r4]
	list_for_each_entry_safe(list, tmp, &head_node, node) {
  64:	e1a04006 	mov	r4, r6
		kfree(list);
  68:	ebfffffe 	bl	0 <kfree>
	list_for_each_entry_safe(list, tmp, &head_node, node) {
  6c:	e5966000 	ldr	r6, [r6]
  70:	eaffffec 	b	28 <cleanup_module+0x28>
	}

	pr_info("hello1 exit\n");
  74:	e3000000 	movw	r0, #0
  78:	e3400000 	movt	r0, #0
}
  7c:	e8bd47f0 	pop	{r4, r5, r6, r7, r8, r9, sl, lr}
	pr_info("hello1 exit\n");
  80:	eafffffe 	b	0 <printk>

```

## Пошук помилки в hello71.ko.unstripped
![image3](image3.png)

## Висновок 
Під час виконання лабораторної роботи було модифіковано програму з 6 лабораторної роботи. Було додано BUG_ON() замість друку повідомлення.
