---
title: DM8168 Uboot support EMAC1
date: 2015-11-11 16:35:44
tags:
- Uboot
- DM8168
---
We only designed a Ethernet interface with Emac1, so I had to modified the Uboot sources from TI.
MII interface is a manager to read the PHY address of network chip when this interface is initialized. The value of the PHY address will be save in a 32-bit register. For example, the address is 3, and the value will be 0×00000008.  The value’s 3rd bit is set as ’1′, so I find the value is 0×08, not 0×03. I have come to the conclusion that a MII interface can manage 32 network chip at most.

Edit *arch/arm/include/asm/arch-ti81xx/emac_defs.h*,
```
#define EMAC_MDIO_PHY_NUM 3

#define EMAC_MDIO_PHY_MASK              (1 < < EMAC_MDIO_PHY_NUM)
 ```
So its mask is 0×08.

Next, change EMAC based address:
```
//#define EMAC0

#ifdef EMAC0

#define EMAC_MDIO_PHY_NUM               (1)
#define EMAC_BASE_ADDR                  (0x4A100000)
#define EMAC_WRAPPER_BASE_ADDR          (0x4A100900)
#define EMAC_WRAPPER_RAM_ADDR           (0x4A102000)

#else

#define EMAC_MDIO_PHY_NUM               (3)
#define EMAC_WRAPPER_RAM_ADDR           (0x4A122000)
#define EMAC_WRAPPER_BASE_ADDR          (0x4A120900)
#define EMAC_BASE_ADDR                  (0x4A120000)

#endif

#define EMAC_MDIO_BASE_ADDR             (0x4A100800)
#define DAVINCI_EMAC_VERSION2
#define DAVINCI_EMAC_GIG_ENABLE

#define EMAC_MDIO_BUS_FREQ              (250000000UL)
#define EMAC_MDIO_CLOCK_FREQ            (2000000UL)
In default configuration, Emac0′s IO is configurated by default, but Emac1′s IO should been configurated extra. Edit drivers/net/davinci_emac.c :

#ifndef EMAC0
#define EMAC1_PINCTRL_BASE       *( volatile unsigned int* )( 0x481408C8 )
/*Initialize EMAC1 pins */
pReg = &amp;EMAC1_PINCTRL_BASE;
for ( i = 0 ; i < 24 ; i++ )
{
debug_emac("Before : 0x%08x\t",readl(pReg));
writel(readl(pReg)|0x01,pReg);
debug_emac("After : 0x%08x\n",readl(pReg));
pReg++;
}
#endif
```
The initialization of the network chip may not been completed  when CPU reads network chip, because CPU speed is too fast. The network will be invalid  in this case.

we can find a method in davinci_emac_initialize().
```bash
for (i = 0; i < 256; i++) {
alive = readl(&amp;adap_mdio->ALIVE);
//      printf("times:%03d,alive:0x%08x\n",i,alive);
if (alive)
break;
udelay(10);
}
```
We can increase time delay to fix this issue.