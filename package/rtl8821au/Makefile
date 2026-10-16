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

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

define KernelPackage/rtl8821au
  SUBMENU:=Wireless Drivers
  TITLE:=Realtek 8821AU/8811AU USB driver (morrownr, + TP-Link T2U Nano 2357:012e)
  DEPENDS:=+kmod-cfg80211 +kmod-usb-core +@DRIVER_11AC_SUPPORT
  FILES:=$(PKG_BUILD_DIR)/8821au.ko
  AUTOLOAD:=$(call AutoProbe,8821au)
endef

define Build/Compile
	+$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) \
		ARCH="$(LINUX_KARCH)" \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		KVER="$(LINUX_VERSION)" \
		KSRC="$(LINUX_DIR)" \
		modules
endef

$(eval $(call KernelPackage,rtl8821au))
