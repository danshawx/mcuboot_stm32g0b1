# mcuboot_stm32g0b1

## flash map
|slot                |flash Area Id     |Start       |End           |Size |
| ------------------ | ---------------- | ---------- | ------------ | --- |
|bootloader          |0                 |0x08000000  |0x0800C800    |50kB |
|app primary Slot    |1                 |0x0800C800  |0x0802C800    |128kB|
|app Upgrade Slot    |2                 |0x0802C800  |0x0804C800    |128kB| 

## Upgrade mode
- overwrite-only configure in mcuboot_config.h:
```
#define MCUBOOT_OVERWRITE_ONLY
#ifdef MCUBOOT_OVERWRITE_ONLY
/* Uncomment to only erase and overwrite those primary slot sectors needed
 * to install the new image, rather than the entire image slot. */
#define MCUBOOT_OVERWRITE_ONLY_FAST
#endif
```

## sign bin
The imgtool sign command can be used to add the MCUboot Image header and trailer to the binary image:

- app primary Slot:

```
python imgtool.py sign --header-size 0x200 --align 4 --slot-size 0x20000 --version 1.0.0 --pad-header C:\xd\demo\my_demo\mcuboot_test\test_bms_boot.bin C:\xd\demo\my_demo\mcuboot_test\test_bms_boot_signed.bin
```

- app Upgrade Slot:
```
python imgtool.py sign --header-size 0x200 --align 4 --slot-size 0x20000 --version 1.0.0 --pad-header C:\xd\demo\my_demo\mcuboot_test\test_bms_boot.bin C:\xd\demo\my_demo\mcuboot_test\test_bms_boot_trailer_signed.bin
```

## choose swap type from application
```
// permanent = 0 swap type:BOOT_SWAP_TYPE_TEST
// permanent = 1 swap type:BOOT_SWAP_TYPE_PERM 
int boot_set_pending(int permanent);
```

## code execution flow
```
boot_go
	context_boot_go
		boot_prepare_image_for_update
			boot_validated_swap_type
				boot_swap_type_multi
                    boot_read_swap_state_by_id  // get boot_swap_state :magic swap_type copy_done image_ok image_num
                        boot_read_swap_state_by_id
                            boot_read_swap_state
```

## other command
- app Upgrade Slot (This slot has image head and image trailer):
```
python imgtool.py sign --header-size 0x200 --align 4 --slot-size 0x20000 --version 1.0.0 --pad-header --pad C:\xd\demo\my_demo\mcuboot_test\test_bms_boot.bin C:\xd\demo\my_demo\mcuboot_test\test_bms_boot_trailer_signed.bin
```
![image trailer](\\app_upgrade_bin_image_trailer.png)

## Reference
[MCUboot Walkthrough and Porting Guide](https://interrupt.memfault.com/blog/mcuboot-overview)
