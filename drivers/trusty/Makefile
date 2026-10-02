# [Fuck4DuerOS] sprd_shm.c 用 #include <trusty.h> 包含本目录头文件。
ccflags-y += -I$(src)

#
# Makefile for trusty components
#

obj-$(CONFIG_TRUSTY) += trusty.o
trusty-y := trusty-core.o trusty-mem.o

obj-$(CONFIG_TRUSTY) += trusty-irq.o
obj-$(CONFIG_TRUSTY) += trusty-pm.o
obj-$(CONFIG_TRUSTY_LOG) += trusty-log.o
obj-$(CONFIG_TRUSTY_VIRTIO) += trusty-virtio.o
obj-$(CONFIG_TRUSTY_VIRTIO_IPC) += trusty-ipc.o
obj-$(CONFIG_SPRD_TSHM) += sprd_shm.o
