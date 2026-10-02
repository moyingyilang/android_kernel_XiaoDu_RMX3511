# [Fuck4DuerOS] sprd_ddr_dvfs_core.c 用 #include <sprd_ddr_dvfs.h>（尖括号）
# 包含同目录头文件，尖括号不会搜索源码目录，需显式加 -I。
ccflags-y += -I$(src)

obj-$(CONFIG_DEVFREQ_SPRD_DDR_DVFS)		+= sprd_ddr_dvfs.o
sprd_ddr_dvfs-m := sprd_ddr_dvfs_core.o
sprd_ddr_dvfs-$(CONFIG_DEVFREQ_SPRD_DDR_DVFS_SP)	+= sprd_ddr_dvfs_sp.o
sprd_ddr_dvfs-$(CONFIG_DEVFREQ_GOV_SPRD_VOTE)		+= sprd_governor_vote.o
