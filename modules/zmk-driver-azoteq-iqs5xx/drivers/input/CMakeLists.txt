# Vendored: zephyr_library_amend() derives the library name from the module root,
# which is this repo rather than the driver directory, so use a named library.
if(CONFIG_INPUT_AZOTEQ_IQS5XX)
  zephyr_library_named(zmk_driver_azoteq_iqs5xx)
  zephyr_library_sources(iqs5xx.c)
endif()
