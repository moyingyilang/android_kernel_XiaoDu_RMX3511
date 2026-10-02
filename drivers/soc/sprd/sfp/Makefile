# [Fuck4DuerOS] sfp_tuple.c 等用 #include <sfp.h> 包含同目录头文件。
ccflags-y += -I$(src)

GCOV_PROFILE = ${CONFIG_SPRD_SFP_GCOV}
obj-$(CONFIG_SPRD_SFP_SUPPORT)  += sfp_core.o

sfp_core-objs := sfp_mgr.o sfp_fwd.o sfp_intf.o sfp_proc.o \
	 sysctl_net_sfp.o sfp_ct.o sfp_hook.o sfp_ipa.o \
	 sfp_tuple.o sfp_test.o
