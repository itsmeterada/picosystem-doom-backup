#tiny_picosystem_executable(
#    monkey
#    monkey.cpp
#)

#pixel_double(monkey)
#disable_startup_logo(snake)

if (PICO_SDK)
  set(BINARIES doom)
else()
  message(SEND_ERROR "Only rp2040 is supported")
endif()

add_subdirectory(adpcm-xq)

# screens
add_subdirectory(pico_st7789)
add_subdirectory(pico_ssd1306)
add_subdirectory(pico_ssd1306_i2c)
add_subdirectory(hagl_hal)
#add_subdirectory(picosystem)

add_subdirectory(doom)

# Common source files used by absolutely everything:
cmake_policy(SET CMP0076 NEW)


if (PICO_SDK)
  set(I_PLATFORM pico)
else()
  message(SEND_ERROR "Only rp2040 is supported")
endif()


add_library(common INTERFACE)
target_sources(common INTERFACE
  i_main.c
  i_system.h
  m_argv.c
  m_argv.h
  m_misc.c
  m_misc.h
)

target_compile_definitions(common INTERFACE
  PICO_XOSC_STARTUP_DELAY_MULTIPLIER=64
  #HACK_FINALE_E1M1=1
  #HACK_FINALE_SHAREWARE=1
  #DEBUG_MOBJ=1
)

if (PICO_SDK)
  add_subdirectory(pico)
else()
  message(SEND_ERROR "Only rp2040 is supported")
endif()

target_link_libraries(common INTERFACE common_${I_PLATFORM})


add_library(game INTERFACE)
target_sources(game INTERFACE
  aes_prng.c
  aes_prng.h
  d_event.c
  d_event.h
  doomkeys.h
  doomtype.h
  d_iwad.c
  d_iwad.h
  d_loop.c
  d_loop.h
  d_mode.c
  d_mode.h
  d_ticcmd.h
  deh_str.c
  deh_str.h
  gusconf.c
  gusconf.h
  i_cdmus.h
  i_endoom.h
  i_glob.h
  i_input.h
  i_joystick.h
  i_swap.h
  i_midipipe.h
  i_sound.h
  i_timer.h
  i_video.h
  i_videohr.h
  midifile.c
  midifile.h
  mus2mid.c
  mus2mid.h
  m_bbox.c
  m_bbox.h
  m_cheat.c
  m_cheat.h
  m_config.c
  m_config.h
  m_controls.c
  m_controls.h
  m_fixed.c
  m_fixed.h

  net_client.c
  net_client.h
  sha1.c
  sha1.h
  memio.c
  memio.h
  tables.c
  tables.h
  v_diskicon.c
  v_diskicon.h
  v_video.c
  v_video.h
  v_patch.h

  w_checksum.c
  w_checksum.h
  w_main.c
  w_main.h
  w_wad.c
  w_wad.h
  w_file.c
  w_file.h
  w_file_posix.c
  w_file_memory.c

  w_merge.c
  w_merge.h
  z_zone.c
  z_zone.h

  i_oplmusic.c
  i_sound.c
)

add_library(game_pico INTERFACE)

target_link_libraries(game INTERFACE game_${I_PLATFORM})
target_link_libraries(game INTERFACE common opl)

add_library(game_deh INTERFACE)
target_sources(game_deh INTERFACE
  deh_defs.h
  deh_io.c
  deh_io.h
  deh_main.c
  deh_main.h
  deh_mapping.c
  deh_mapping.h
  deh_text.c
)
target_link_libraries(game_deh INTERFACE game)

add_library(small_doom_common INTERFACE)
target_link_libraries(small_doom_common INTERFACE adpcm-lib) # handy to have about

target_compile_definitions(small_doom_common INTERFACE
        SHRINK_MOBJ=1 # may break saved games

        DOOM_ONLY=1 # don't support hexxen etc.
        DOOM_SMALL=1
        DOOM_CONST=1 # thread const thru lots of places

        SOUND_LOW_PASS=1
        NUM_SOUND_CHANNELS=8 # sounds ok (actually that is how many are used by default)

        # functionality
        NO_USE_CHECKSUM=1
        NO_USE_RELOAD=1
        USE_SINGLE_IWAD=1
        NO_USE_WIPE=1 #memory if nothing else - we can fix this (you can do it)
        NO_USE_JOYSTICK=1
        NO_USE_DEH=1
        NO_USE_MUSIC_PACKS=1

        USE_FLAT_MAX_256=1
        USE_MEMMAP_ONLY=1

        USE_LIGHTMAP_INDEXES=1 # saves about 9K

        #FRACBITS=15 # ha ha doesn't seem to be passed thru everywhere!
        NO_USE_ENDDOOM=1
        #NO_DRAW_VISPLANES=1

        USE_ERASE_FRAME=1 #
        NO_DRAW_MID=1
        NO_DRAW_TOP=1
        NO_DRAW_BOTTOM=1
        NO_DRAW_MASKED=1
        NO_DRAW_SKY=1
        #NO_DRAW_SPANS=1
        NO_DRAW_SPRITES=1
        NO_DRAW_PSPRITES=1

#        NO_VISPLANES=1
        NO_VISPLANE_GUTS=1
        NO_VISPLANE_CACHES=1 # todo these might be fine as they can be temporary
        NO_DRAWSEGS=1
        NO_VISSPRITES=1


        NO_MASKED_FLOOR_CLIP=1 # not needed with PD_ rendering and with floor clip it we have occasional visual glitches with sprites split across visplanes

        PD_DRAW_COLUMNS=1
        PD_DRAW_MARKERS=1
        PD_DRAW_PLANES=1

        PD_SCALE_SORT=1

        PD_CLIP_WALLS=1
        PD_QUANTIZE=1

        PD_SANITY=1

        #MU_STATS=1
        PD_COLUMNS=1
        PICO_DOOM=1
        NO_USE_DS_COLORMAP=1
        NO_USE_DC_COLORMAP=1

        #PRINT_COLORMAPS=1
        #PRINT_PALETTE=1
        USE_READONLY_MMAP=1

# -----------------------------------------------------------------
# MUSIC RELATED
# -----------------------------------------------------------------
        NO_USE_TIMIDITY=1
        NO_USE_GUS=1
        NO_USE_LIBSAMPLERATE=1

        # slightly slower but only uses 1K of sin tables vs 9K
        EMU8950_NO_WAVE_TABLE_MAP=1

        EMU8950_NO_TLL=1 # don't use lookup table for total level
        EMU8950_NO_FLOAT=1 # double check there is no float
        EMU8950_NO_TIMER=1 # disable timer which isn't used
        EMU8950_NO_TEST_FLAG=1 # disable test flags (which aren't used)
        EMU8950_SIMPLER_NOISE=1 # only generate noise bit when needed
        EMU8950_SHORT_NOISE_UPDATE_CHECK=1 # only update short noise if it is used
        # actually this doesn't make things faster
        #EMU8950_LINEAR_NEG_NOT_NOT=1 # negative values rounded towrds -infinity not 0 without this; does it matter?

        EMU8950_LINEAR_SKIP=1 # skip silent slots
        EMU8950_LINEAR_END_OF_NOTE_OPTIMIZATION # early out envelope when DECAY/SUSTAIN/RELEASE envelope reaches mute
        EMU8950_NO_PERCUSSION_MODE=1 # rhythm only mode (doom doesn't use percussion; whhhaaaah!?)
        EMU8950_LINEAR=1 # reorganize to do linear runs of channels

        # things we really don't care about but need for diff-ing with non linear version
        # BEWARE - need to turn this off for comparison with on device version which doesn't do nitpicks (though could i guess)

        EMU8950_ASM=1
#        EMU8950_NIT_PICKS=1
#        DUMPO=1

# -----------------------------------------------------------------
# FLASH SIZE
# -----------------------------------------------------------------
        NO_USE_STATE_MISC #doesn't appear to be used anyway - perhaps in doom only

# -----------------------------------------------------------------
# RAM SIZE
# -----------------------------------------------------------------
        USE_RAW_MAPNODE=1
        USE_RAW_MAPVERTEX=1
        USE_RAW_MAPSEG=1
        USE_RAW_MAPLINEDEF=1
        # USE_RAW_MAPSUBSECTOR=1 # obsolete... now WHD
        # USE_RAW_MAPSIDEDEF=1 obsolete
        USE_RAW_MAPTHING=1

        USE_INDEX_LINEBUFFER=1
        NO_USE_ZLIGHT=1
        NO_Z_ZONE_ID=1 # seems unused

        Z_MALOOC_EXTRA_DATA=1
        USE_THINKER_POOL=1
        NO_INTERCEPTS_OVERRUN=1
#        INCLUDE_SOUND_C_IN_S_SOUND=1 # avoid issues with non static const array
# -----------------------------------------------------------------
# IMMUTABLE
# -----------------------------------------------------------------

        TEMP_IMMUTABLE_DISABLED=1
        USE_CONST_SFX=1
        USE_CONST_MUSIC=1

        NO_DEMO_RECORDING=1
        PICO_NO_TIMING_DEMO=1
        NO_USE_EXIT=1 # not sure whether we have an exit... if so what does it need to do?

        PICO_DEBUG_PIN_BASE=18
        PICO_DEBUG_PIN_COUNT=2
)

target_compile_options(small_doom_common INTERFACE
  -Wall
  -Wno-unused-function
  -Wno-unused-but-set-variable
  -Wno-unused-variable
)

if (TARGET chocolate-doom)
  target_compile_definitions(chocolate-doom PRIVATE USE_FLAT_MAX_256=1)
endif()


function(add_doom_tiny SUFFIX SCREEN)
  add_executable(doom_tiny${SUFFIX})
  target_link_libraries(doom_tiny${SUFFIX} PRIVATE game)

  target_sources(doom_tiny${SUFFIX} PRIVATE
    tiny_huff.c
    musx_decoder.c
    image_decoder.c
  )
  target_include_directories(doom_tiny${SUFFIX} PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/../")
  target_link_libraries(doom_tiny${SUFFIX} PRIVATE doom ${EXTRA_LIBS})
  target_link_libraries(doom_tiny${SUFFIX} PRIVATE small_doom_common)

  # pico_enable_stdio_usb(doom_tiny${SUFFIX} 0)
  # pico_enable_stdio_uart(doom_tiny${SUFFIX} 1)

  if (PICO_SDK)
    pico_set_float_implementation(doom_tiny${SUFFIX} none)
    pico_set_double_implementation(doom_tiny${SUFFIX} none)
    pico_add_extra_outputs(doom_tiny${SUFFIX})
  endif()

  target_compile_options(doom_tiny${SUFFIX} PRIVATE -Wno-format-truncation)
  target_compile_definitions(doom_tiny${SUFFIX} PRIVATE
    DOOM_TINY=1

    # LILYGO_TTGO=1
    # ST7789_240_135=1
    # SSD1306_70_40=1
    # SSD1306_70_40_i2c=1
    ${SCREEN}=1
        
    SUPPORT_OVERLAYS=1
    EMU8950_SLOT_RENDER=1

    SHRINK_MOBJ=1 # may break saved games
    NO_RDRAW=1

    #NO_DRAW_MENU=1
    # SDK minimilization
    PICO_TIME_DEFAULT_ALARM_POOL_DISABLED=1
    PICO_TIME_DEFAULT_ALARM_POOL_MAX_TIMERS=1
    PICO_DISABLE_SHARED_IRQ_HANDLERS=1
    PICO_USE_OPTIMISTIC_SBRK=1

    # MUSIC
    USE_EMU8950_OPL=1
    USE_DIRECT_MIDI_LUMP=1

    NO_USE_NET=1 # standard networking
    USE_PICO_NET=0
    # for vgaboard
    # PICO_DEFAULT_I2C=1
    # PICO_DEFAULT_I2C_SDA_PIN=18
    # PICO_DEFAULT_I2C_SCL_PIN=19

    #NO_USE_LOAD=1
    #NO_USE_SAVE=1
    $<$<BOOL:${PICO_ON_DEVICE}>:NO_FILE_ACCESS=1>

    SAVE_COMPRESSED=1
    LOAD_COMPRESSED=1

    NO_USE_ARGS=1
    NO_USE_SAVE_CONFIG=1
    NO_USE_FLOAT=1

    USE_VANILLA_KEYBOARD_MAPPING_ONLY=1

    #FORCE_NODRAW=1

    NO_USE_LOADING_DISK=1

    # -- these aren't used any more, we adapt based on DOOM_TINY
    #NO_USE_ST=1 # for now skip all the status crap
    #NO_USE_DEFAULT_ST_LIB=1

    #NO_PAGE_DRAWER=1 # splash screens
    #NO_HU_DRAWER=1

    USE_WHD=1

    NO_Z_MALLOC_USER_PTR=1 # onyl needed for freeing textures i think

    FIXED_SCREENWIDTH=1
    FLOOR_CEILING_CLIP_8BIT=1

    # NO_USE_WI=1
    USE_MUSX=1
    MUSX_COMPRESSED=1

    NO_SCREENSHOT=1

    NO_USE_BOUND_CONFIG=1

    USE_FPS=1

    # PICO_DOOM_INFO=1
  )

  target_compile_definitions(doom_tiny${SUFFIX} PRIVATE
    #FORCE_NODRAW=1
    USE_MEMORY_WAD=1
    PICO_DEBUG_MALLOC=1
    EMU8950_NO_RATECONV=1

    PICO_CORE1_STACK_SIZE=0x4f8

    NO_IERROR=1
  )

  if (PICO_ON_DEVICE)
    # target_link_libraries(doom_tiny${SUFFIX} PRIVATE hardware_spi)
    # TODO use defs or targets for these?
    target_link_libraries(doom_tiny${SUFFIX} PRIVATE pico_st7789 hardware_spi)
    target_link_libraries(doom_tiny${SUFFIX} PRIVATE pico_ssd1306)
    target_link_libraries(doom_tiny${SUFFIX} PRIVATE pico_ssd1306_i2c hardware_i2c)
    target_link_libraries(doom_tiny${SUFFIX} PRIVATE hagl_hal hardware_spi)
    target_compile_options(doom_tiny${SUFFIX} PRIVATE -fno-common -fdata-sections -Wl,--sort-section=alignment)
    target_compile_definitions(doom_tiny${SUFFIX} PRIVATE NO_ZONE_DEBUG=1)
  endif()
  if (FORCE_DEBUG)
    target_compile_options(doom_tiny${SUFFIX} PRIVATE -g)
  endif()
  target_link_libraries(doom_tiny${SUFFIX} PRIVATE render_newhope)
  set(PICO_HACK 1)
  set(STAMP_HACK 0)
  if (PICO_HACK)
    target_compile_definitions(doom_tiny${SUFFIX} PRIVATE
      PICO_AUDIO_I2S_DATA_PIN=20
      PICO_AUDIO_I2S_CLOCK_PIN_BASE=21
      NO_USE_UART=1
    )
    pico_enable_stdio_uart(doom_tiny${SUFFIX} 0)
  elseif(STAMP_HACK)
    target_compile_definitions(doom_tiny${SUFFIX} PRIVATE
      PICO_AUDIO_I2S_DATA_PIN=22
      PICO_AUDIO_I2S_CLOCK_PIN_BASE=23
      NO_USE_UART=1
      INCREASE_I2S_DRIVE_STRENGTH=1
    )
    pico_enable_stdio_uart(doom_tiny${SUFFIX} 0)
  endif()
endfunction()

add_subdirectory(whd_gen)

add_library(render_newhope INTERFACE)
target_sources(render_newhope INTERFACE pd_render.cpp)
target_compile_definitions(render_newhope INTERFACE
  PICODOOM_RENDER_NEWHOPE=1
  MERGE_DISTSCALE0_INTO_VIEWCOSSINANGLE=1
  NO_USE_ZLIGHT=1
  PICO_SCANVIDEO_SCANLINE_BUFFER_COUNT=4
  PICO_SCANVIDEO_LINKED_SCANLINE_BUFFERS=1
  PICO_SCANVIDEO_SCANLINE_RELEASE_FUNCTION=1
)
if (PICO_ON_DEVICE)
  target_compile_definitions(render_newhope INTERFACE PICO_SCANVIDEO_MAX_SCANLINE_BUFFER_WORDS=164)
else()
  message(SEND_ERROR "Only rp2040 is supported")
endif()

if (PICO_SDK)
  add_library(tiny_settings INTERFACE)
  target_compile_definitions(tiny_settings INTERFACE
    WHD_SUPER_TINY=1
    DEMO1_ONLY=1
    NO_USE_FINALE_CAST=1
    NO_USE_FINALE_BUNNY=1
  )
endif()


function(add_doom_tiny_screen SCREEN)
  if (PICO_SDK)
    # add_doom_tiny(_${SCREEN} ${SCREEN})

    # target_compile_definitions(doom_tiny_${SCREEN} PRIVATE
    #         TINY_WAD_ADDR=0x10040000
    # )
    # target_link_libraries(doom_tiny_${SCREEN} PRIVATE tiny_settings)

    # disabled because I don't need it, probably still works...
    # add_doom_tiny(_nost_${SCREEN} ${SCREEN})

    # target_compile_definitions(doom_tiny_nost_${SCREEN} PRIVATE
    #         # NO_USE_FINALE_CAST=1 # note this is only used by doom II but doesnt take that much more space
    #         TINY_WAD_ADDR=0x10048000
    #         )
    if (PICO_ON_DEVICE)
      # suppress_tinyusb_warnings()
      # add_doom_tiny(_usb_${SCREEN} ${SCREEN})
      # target_link_libraries(doom_tiny_usb_${SCREEN} PRIVATE tiny_settings pico_cd)
      # target_compile_definitions(doom_tiny_usb_${SCREEN} PRIVATE 
      #   USB_SUPPORT=1
      #   TINY_WAD_ADDR=0x10042000 # ugh tinyusb is >8K currently!!
      # )

      add_doom_tiny(_${SCREEN} ${SCREEN})
      target_link_libraries(doom_tiny_${SCREEN} PRIVATE tiny_settings)
      target_compile_definitions(doom_tiny_${SCREEN} PRIVATE TINY_WAD_ADDR=0x10040000)

      # disabled for same reason as above
      # add_doom_tiny(_nost_usb_${SCREEN} ${SCREEN})
      # target_link_libraries(doom_tiny_nost_usb_${SCREEN} PRIVATE pico_cd)
      # target_compile_definitions(doom_tiny_nost_usb_${SCREEN} PRIVATE
      #         USB_SUPPORT=1
      #         TINY_WAD_ADDR=0x10048000
      #         )
    endif()
  endif()
endfunction()


# add_doom_tiny_screen("SSD1306_70_40")
# target_compile_definitions(doom_tiny_usb_SSD1306_70_40 PRIVATE
#   DOWNSAMPLING_FACTOR_OUT_OF_100=500
# )

add_doom_tiny_screen("SSD1306_70_40_i2c")
target_compile_definitions(doom_tiny_SSD1306_70_40_i2c PRIVATE
  DOWNSAMPLING_FACTOR_OUT_OF_100=500
)

# add_doom_tiny_screen("LILYGO_TTGO")
# target_compile_definitions(doom_tiny_usb_LILYGO_TTGO PRIVATE
#   DOWNSAMPLING_FACTOR_OUT_OF_100=148
# )

# add_doom_tiny_screen("ST7789_240_135")
# target_compile_definitions(doom_tiny_usb_ST7789_240_135 PRIVATE
#   DOWNSAMPLING_FACTOR_OUT_OF_100=148
# )

# add_doom_tiny_screen("ST7735_128_128")
# target_compile_definitions(doom_tiny_usb_ST7735_128_128 PRIVATE
#   DOWNSAMPLING_FACTOR_OUT_OF_100=250

#   # hagl-hal stuff
   
#   HAGL_HAL_USE_SINGLE_BUFFER
#   # MIPI_DISPLAY_INVERT=1

#   # screen-specific
#   MIPI_DISPLAY_PIN_DC=1
#   MIPI_DISPLAY_PIN_CS=5
#   MIPI_DISPLAY_PIN_CLK=2
#   MIPI_DISPLAY_PIN_MOSI=3
#   MIPI_DISPLAY_PIN_RST=0
#   MIPI_DISPLAY_PIN_BL=22
#   MIPI_DISPLAY_PIN_MISO=-1

#   MIPI_DISPLAY_SPI_PORT=spi0
#   MIPI_DISPLAY_SPI_CLOCK_SPEED_HZ=62500000
#   MIPI_DISPLAY_PIXEL_FORMAT=MIPI_DCS_PIXEL_FORMAT_16BIT

#   # MIPI_DISPLAY_ADDRESS_MODE=MIPI_DCS_ADDRESS_MODE_BGR|MIPI_DCS_ADDRESS_MODE_MIRROR_Y|MIPI_DCS_ADDRESS_MODE_MIRROR_X
#   MIPI_DISPLAY_ADDRESS_MODE=MIPI_DCS_ADDRESS_MODE_BGR|MIPI_DCS_ADDRESS_MODE_SWAP_XY|MIPI_DCS_ADDRESS_MODE_MIRROR_Y
#   MIPI_DISPLAY_WIDTH=160
#   MIPI_DISPLAY_HEIGHT=80
#   MIPI_DISPLAY_OFFSET_X=0
#   MIPI_DISPLAY_OFFSET_Y=26

#   # MIPI_DISPLAY_WIDTH=128
#   # MIPI_DISPLAY_HEIGHT=128
#   # MIPI_DISPLAY_OFFSET_X=2
#   # MIPI_DISPLAY_OFFSET_Y=2

#   MIPI_DISPLAY_INVERT=1
# )
