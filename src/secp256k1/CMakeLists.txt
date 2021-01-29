message("I'm /src/secp256k1")

set(WITH_BIGNUM auto CACHE STRING "bignum implementation to use [default=auto]")
set_property(CACHE WITH_BIGNUM PROPERTY STRINGS auto no gmp)

set(WITH_ECMULT_WINDOW auto CACHE STRING "window size for ecmult precomputation for verification, specified as integer in range [2..24]. Larger values result in possibly better performance at the cost of an exponentially larger precomputed table. The table will store 2^(SIZE-1) * 64 bytes of data but can be larger in memory due to platform-specific padding and alignment. \"auto\" is a reasonable setting for desktop machines (currently 15). [default=auto]")
set_property(CACHE WITH_ECMULT_WINDOW PROPERTY STRINGS auto 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24)

set(WITH_ECMULT_GEN_PRECISION auto CACHE STRING "Precision bits to tune the precomputed table size for signing. The size of the table is 32kB for 2 bits, 64kB for 4 bits, 512kB for 8 bits of precision. A larger table size usually results in possible faster signing. \"auto\" is a reasonable setting for desktop machines (currently 4). [default=auto]")
set_property(CACHE WITH_ECMULT_GEN_PRECISION PROPERTY STRINGS auto 2 4 8)

add_library(libsecp256k1 src/secp256k1.c)
target_include_directories(libsecp256k1 PUBLIC include . PRIVATE src)

if("${WITH_BIGNUM}" STREQUAL "gmp") # todo: condition for gmp
  find_package(gmp REQUIRED)
  set(USE_BIGNUM YES)
elseif("${WITH_BIGNUM}" STREQUAL "no")
  set(USE_BIGNUM NO)
elseif("${WITH_BIGNUM}" STREQUAL "auto")
  find_package(gmp)
  if(gmp_NOTFOUND)
    set(USE_BIGNUM NO)
  else()
    set(USE_BIGNUM YES)
  endif()
else()
  message(FATAL_ERROR "invalid bignum implementation")
endif()

if(USE_BIGNUM)
  target_compile_definitions(libsecp256k1 PRIVATE
    USE_NUM_NONE=1
    USE_FIELD_INV_BUILTIN=1
    USE_SCALAR_INV_BUILTIN=1)
else()
  target_compile_definitions(libsecp256k1 PRIVATE
    HAVE_LIBGMP=1
    USE_NUM_GMP=1
    USE_FIELD_INV_NUM=1
    USE_SCALAR_INV_NUM=1)
endif()

if("${WITH_ECMULT_WINDOW}" STREQUAL "auto")
  set(ECMULT_WINDOW_SIZE 15)
else()
  set(ECMULT_WINDOW_SIZE ${WITH_ECMULT_WINDOW})
endif()

target_compile_definitions(libsecp256k1 PRIVATE ECMULT_WINDOW_SIZE=${ECMULT_WINDOW_SIZE})

if("${WITH_ECMULT_GEN_PRECISION}" STREQUAL "auto")
  set(ECMULT_GEN_PREC_BITS 4)
else()
  set(ECMULT_GEN_PREC_BITS ${WITH_ECMULT_GEN_PRECISION})
endif()

target_compile_definitions(libsecp256k1 PRIVATE ECMULT_GEN_PREC_BITS=${ECMULT_GEN_PREC_BITS})


