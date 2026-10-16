include $(TOPDIR)/rules.mk

PKG_NAME:=rtl8821au
PKG_VERSION:=5.12.5.2
PKG_RELEASE:=1

PKG_SOURCE:=8821au-20210708-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/morrownr/8821au-20210708/tar.gz/1a819991f5b75e64dfcf922b96a6681f367cbba0?
PKG_HASH:=c97877fd86c051276e67ddbb4deee42fc59e7fe99d87a0f20e6aff53da210f38
PKG_BUILD_DIR:=$(BUILD_DIR)/8821au-20210708-1a819991f5b75e64dfcf922b96a6681f367cbba0

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_PARALLEL:=1

STAMP_CONFIGURED_DEPENDS := $(STAGING_DIR)/usr/include/mac80211-backport/backport/autoconf.h

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

define KernelPackage/rtl8821au
  SUBMENU:=Wireless Drivers
  TITLE:=Realtek 8821AU/8811AU USB driver (morrownr, + TP-Link T2U Nano 2357:012e)
  DEPENDS:=+kmod-cfg80211 +kmod-usb-core +@DRIVER_11AC_SUPPORT
  FILES:=$(PKG_BUILD_DIR)/8821au.ko
  AUTOLOAD:=$(call AutoProbe,8821au)
endef

NOSTDINC_FLAGS := \
	$(KERNEL_NOSTDINC_FLAGS) \
	-I$(PKG_BUILD_DIR) \
	-I$(PKG_BUILD_DIR)/include \
	-I$(STAGING_DIR)/usr/include/mac80211-backport \
	-I$(STAGING_DIR)/usr/include/mac80211-backport/uapi \
	-I$(STAGING_DIR)/usr/include/mac80211 \
	-I$(STAGING_DIR)/usr/include/mac80211/uapi \
	-include backport/backport.h \
	-Wno-error

define Build/Compile
	+$(MAKE) -C $(PKG_BUILD_DIR) clean >/dev/null 2>&1 || true
	+$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) \
		ARCH="$(LINUX_KARCH)" \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		KVER="$(LINUX_VERSION)" \
		KSRC="$(LINUX_DIR)" \
		USER_EXTRA_CFLAGS="$(NOSTDINC_FLAGS)" \
		KBUILD_EXTRA_SYMBOLS="$(KERNEL_BUILD_DIR)/symvers/mac80211.symvers" \
		modules
endef

$(eval $(call KernelPackage,rtl8821au))
