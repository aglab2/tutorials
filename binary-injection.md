# Binary Injection
This guide describes how binary injection works to interop C code with N64 MIPS asm. It is highly recommended to have the basic knowledge of C to understand this. Knowledge of binary hacking ASM is a useful addition although I do understand it is very complicated subject. It is recommended to setup PJ64 3.x Debugger which is outside of this tutorial scope. In general binary hacking with C injection is easier for business logic but setting it up can be cumbersome. I only cover the very basics of this process, more work to adjust the code will be absolutely necessary so have some patience when working with it. A great alternative to this guide is using [HackerSM64](https://github.com/HackerN64/HackerSM64) which is much nicer to work with. Really, if you plan to do some crazy patches in C, you are up for a rough time with C injection as this guide will likely show.

# Tools
* [Cmder](https://github.com/cmderdev/cmder/releases/latest)
* [LLVM Clang](https://github.com/llvm/llvm-project/releases). Windows installer has name "LLVM-18.1.6-win64.exe" or similar.
* [make](https://gnuwin32.sourceforge.net/downlinks/make-bin-zip.php). Extract in cmder folder in "bin".

Unpack cmder anywhere and install clang. Make sure to select "Add LLVM to the system PATH for all users".

Execute cmder and check that clang/clang++ works:
```
λ clang --version
clang version 18.1.6
Target: x86_64-pc-windows-msvc
Thread model: posix
InstalledDir: C:\Program Files\LLVM\bin

λ clang++ --version
clang version 18.1.6
Target: x86_64-pc-windows-msvc
Thread model: posix
InstalledDir: C:\Program Files\LLVM\bin
```

If for some reason it says "' is not recognized as an internal or external command, operable program or batch file.", add clang to PATH manually. In search bar search for "Edit the system environment variables". Click on "Environment Variables...". in "System Variables" double click on "Path" entry. Click "New", then "Browse" and select "C:\Program Files\LLVM\bin" or other folder where you installed LLVM.

# C code compilation

I will use Decades Later source codes as an example. Usually binary injections work the same way but there is obviously some ROM specific shenanigans.

Code is located in [GitHub repo](https://github.com/aglab2/sm64asm/tree/master/dl).
You will also need [decomp API headers](https://github.com/aglab2/sm64-api). Put it in "dl" folder. 

If you want to apply this code, you must turn on in "Music table" tab option "Enable overwrite size restriction".
In cmder navigate to folder where dl is downloaded/cloned. Name rom where you will apply changes as "dl.z64" and place it outside of "dl" folder. For example source code is in "D:\git\sm64asm\dl", ROM should be in "D:\git\sm64asm\dl.z64", API should be in "D:\git\sm64asm\dl\sm64-api".
Execute `make`, it should compile the sources and write the compiled payload in ROM.

Here is an example output and commands to navigate to folder with source codes.
```
λ D:
λ cd D:\git\sm64asm\dl
D:\git\sm64asm\dl (master -> origin)
λ make
mkdir obj
clang++ -DVERSION_US -DF3D_OLD=1 -DBINARY -flto -Wall -Wdouble-promotion -Os -mfix4300 -march=mips2 --target=mips-img-elf -fomit-frame-pointer -G0 -I sm64-api/sm64 -I sm64-api/sm64/libc -mno-check-zero-division -fno-exceptions -fno-builtin -fno-rtti -fno-common -mno-abicalls -DTARGET_N64 -mfpxx src/main.cpp -c -o obj/main.o
...
clang++ -DVERSION_US -DF3D_OLD=1 -DBINARY -flto -Wall -Wdouble-promotion -Os -mfix4300 -march=mips2 --target=mips-img-elf -fomit-frame-pointer -G0 -I sm64-api/sm64 -I sm64-api/sm64/libc -mno-check-zero-division -fno-exceptions -fno-builtin -fno-rtti -fno-common -mno-abicalls -DTARGET_N64 -mfpxx src/camera_y.cpp -c -o obj/camera_y.o
ld.lld -o obj/payload -L. -L sm64-api/ld --oformat binary -T ldscript -M obj/main.o obj/endscreen_blocker.o obj/star_radar.o obj/omm.o obj/omm_geo.o obj/change_music.o obj/green_sb_resetter.o obj/blue_star_block.o obj/puffer.o obj/fail_warp.o obj/ex_objects.o obj/blue_stars_compat.o obj/act_select.o obj/stardisplay.o obj/menudraw.o obj/wall_cucking.o obj/whomp_king.o obj/blue_star_mode.o obj/sound_farewell.o obj/bully_lava_death.o obj/camera_y.o > out.txt
dd bs=1 seek=66912256 if=obj/payload of=../dl.z64 conv=notrunc
25856+0 records in
25856+0 records out
25856 bytes (26 kB, 25 KiB) copied, 0.0534761 s, 484 kB/s
```

# Payload loading

Now as payload is in the ROM, it needs to be copied in RAM. In case of Decades Later, hook is in file [hook2.asm](https://github.com/aglab2/sm64asm/blob/master/dl/hook2.asm). Apply it to the ROM using [Simple Armips GUI](https://github.com/DavidSM64/SimpleArmipsGui/releases/download/1.3.2/Simple.Armips.Gui.v1.3.2.zip). You can check that payload was copied in RAM using Debugger in PJ64 3.x. Use Debugger > Memory > View. Navigate to address 801CE0000 - you should see payload code at that place.

# Hooks to payload

Payload is now loaded in the RAM so ASM can reference it. `_start` symbol is pinned at the start of the payload at `0x801ce000` using [linker script](https://github.com/aglab2/sm64asm/blob/master/dl/ldscript#L7-L9) so RAM addresses of the compiled payload are consistent across code changes. From C code I leave the [comments](https://github.com/aglab2/sm64asm/blob/master/dl/src/main.cpp#L56-L94) helping me know where particular symbol is referenced.
This setup allows for 2 ways to reference compiled ASM code: direct and indirect.

Indirect references are used for direct calls from ASM. I use `JALR` to well known address to load function pointer and call it. An example can be seen in [hook-actselect.asm](https://github.com/aglab2/sm64asm/blob/master/dl/hook-actselect.asm).
```
	LW V1, 0x801ce078
	JALR V1
	NOP
```

This code will end up calling function `actSelectPath`.

Direct refences are usually used for behavior scripts and geolayouts. For example `ChangeMusic` object is defined at (_start)[https://github.com/aglab2/sm64asm/blob/master/dl/src/main.cpp#L68}, it has behavior address `001ce040` (segmented in bank 0).

# HUD render example

It requires a certain level of understanding to realize which hooks need to be compiled in order to add certain functionality to the code. A common way to look at names in `_start`. A good example can be `renderHud` which is indirectly referenced at `801ce01c`. `801ce01c` appears in [hook-hud](https://github.com/aglab2/sm64asm/blob/master/dl/hook-hud.asm) so you need to compile hook-hud.asm to make calls to `renderHud`.

Unfortunately, if you try to compile it, it will start outputting lots of gibberish, it is due to Decades Later using [blue stars](https://github.com/aglab2/sm64asm/blob/master/dl/src/star_radar.cpp#L455-L456) for calculating star counts. You may replace these conditions with temporary stubs to render. Change `renderHud` to remove code referring to blue stars:
```
void renderHud(s32 renderCoins)
{
    // gHudDisplay.stars = _save_file_get_total_star_count(gCurrSaveFileNum - 1, 0, 24);
    s32 blueStarsCount = 0; // _save_file_get_total_star_count(0x100 | (gCurrSaveFileNum - 1), 0, 24);
    if (renderCoins)
    {
        int yoff = blueStarsCount ? 20 : 0;
        print_text(0x16, -yoff + 0xbd, "+"); // 'Coin' glyph
        print_text(0x26, -yoff + 0xbd, "*"); // 'X' glyph
        print_text_fmt_int(0x36, -yoff + 0xbd, "%d", gHudDisplay.coins);
    }

    // renders the radar in bottom left corner, not render when all stars are collected because it is pointless
    // if (gMarioStates->numStars < 333)
        renderStarRadar();
...
```

Modify `renderStarRadar` to remove checks for blue stars existance:
```
void renderStarRadar()
{
    if (!gMarioObject)
        return;

    if (sLastCheckedLevel != gCurrLevelNum)
    {
        sRenderStarRadar = hasAnyWithBparam4((BehaviorScript*) 0x0000000013003e3c);
        sLastCheckedLevel = gCurrLevelNum;
    }

    // if (!sRenderStarRadar)
    //    return;
```

Allow finding non-blue stars in `obj_find_nearest_object_with_behavior_and_bparam1` (remove check for bparam4)
```
static struct Object *obj_find_nearest_object_with_behavior_and_bparam1(const BehaviorScript *behavior, int bparam1) {
    uintptr_t *behaviorAddr = (uintptr_t *) segmented_to_virtual(behavior);
    struct ObjectNode *listHead = &gObjectLists[get_object_list_from_behavior(behaviorAddr)];
    struct Object *obj = (struct Object *) listHead->next;
    while (obj != (struct Object *) listHead) {
        if (obj->behavior == behaviorAddr
            && obj->activeFlags != 0
            && (((obj->oBehParams) >> 24) & 0xff) == bparam1
        ) {
            return obj;
        }

        obj = (struct Object *) obj->header.next;
    }

    return nullptr;
}
```

If you go to BOB, you will see that gibberish textures are being drawn notifying that HUD is rendered although proper textures do not exist yet. Textures are referenced in,
```
    Texture* grayTexture = (Texture*) 0x04078200;
    Texture* yellowTexture = (Texture*) 0x0407CA00;
    Texture* redTexture = (Texture*) 0x0407EE00;
    Texture* blueTexture = (Texture*) 0x0407A600;
...
                            tex = (Texture*) 0x04077E00;
```

Change it to something where textures do exist, for example 0x02000000. Observe that now if you enter BOB, textures are indeed changing according to star radar configs.

To use your own textures, I can recommend expanding the bank 4 similar to Decades Later approach. Bank 4 loading command is at `002ABCA0`. Copy the bank from vanilla location (2nd and 3rd ints, in my case 0x823B70 to 0x858EE8) to 0x3f00000. Change 2nd and 3rd int to 0x3f00000 and 0x3f800000. Now you have extended bank 4 - you can write to empty space after copy end. Segmented address `0x04078200` corresponds to ROM address `0x3f78200`.
