# CMakeLists.txt file for the FCVODE library

# Add variable fcvode_SOURCES with the sources for the FCVODE library
SET(fcvode_SOURCES
  fcvband.c
  fcvbbd.c
  fcvbp.c
  fcvdense.c
  fcvewt.c
  fcvjtimes.c
  fcvnulllinsol.c
  fcvnullmatrix.c
  fcvode.c
  fcvpreco.c
  fcvroot.c
  fcvsparse.c
  )

# Add variable shared_SOURCES with the common SUNDIALS sources which will
# also be included in the CVODE library
SET(shared_SOURCES
  ${sundials_SOURCE_DIR}/src/nvec_ser/fnvector_serial.c
  )

# Add variable sunmatrix_SOURCES with the common SUNMatrix sources which will
# also be included in the CVODE library
SET(sunmatrix_SOURCES
  ${sundials_SOURCE_DIR}/src/sunmat_band/fsunmatrix_band.c
  ${sundials_SOURCE_DIR}/src/sunmat_dense/fsunmatrix_dense.c
  ${sundials_SOURCE_DIR}/src/sunmat_sparse/fsunmatrix_sparse.c
  )

# Add variable sunlinsol_SOURCES with the common SUNLinearSolver sources which will
# also be included in the CVODE library
SET(sunlinsol_SOURCES
  ${sundials_SOURCE_DIR}/src/sunlinsol_band/fsunlinsol_band.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_dense/fsunlinsol_dense.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_spbcgs/fsunlinsol_spbcgs.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_spfgmr/fsunlinsol_spfgmr.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_spgmr/fsunlinsol_spgmr.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_sptfqmr/fsunlinsol_sptfqmr.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_pcg/fsunlinsol_pcg.c
  )

IF(KLU_FOUND)
  LIST(APPEND sunlinsol_SOURCES
    ${sundials_SOURCE_DIR}/src/sunlinsol_klu/fsunlinsol_klu.c
    )
ENDIF()

IF(SUPERLUMT_FOUND)
  LIST(APPEND sunlinsol_SOURCES
    ${sundials_SOURCE_DIR}/src/sunlinsol_superlumt/fsunlinsol_superlumt.c
    )
ENDIF()

IF(LAPACK_FOUND)
  LIST(APPEND sunlinsol_SOURCES
    ${sundials_SOURCE_DIR}/src/sunlinsol_lapackband/fsunlinsol_lapackband.c
    ${sundials_SOURCE_DIR}/src/sunlinsol_lapackdense/fsunlinsol_lapackdense.c
    )
ENDIF()


# Add source directories to include directories for access to
# implementation only header files (both for fcvode and cvode)
INCLUDE_DIRECTORIES(.)
INCLUDE_DIRECTORIES(..)

# Define C preprocessor flag -DBUILD_SUNDIALS_LIBRARY 
ADD_DEFINITIONS(-DBUILD_SUNDIALS_LIBRARY)

# Only build STATIC libraries (we cannot build shared libraries 
# for the FCMIX interfaces due to unresolved symbol errors 
# coming from inexistent user-provided functions)

# Add the build target for the FCVODE library
ADD_LIBRARY(sundials_fcvode_static STATIC
  ${fcvode_SOURCES} ${shared_SOURCES} ${sunmatrix_SOURCES} ${sunlinsol_SOURCES})

# Set the library name and make sure it is not deleted
SET_TARGET_PROPERTIES(sundials_fcvode_static
  PROPERTIES OUTPUT_NAME sundials_fcvode CLEAN_DIRECT_OUTPUT 1)

# Install the FCVODE library
INSTALL(TARGETS sundials_fcvode_static DESTINATION ${CMAKE_INSTALL_LIBDIR})

#
MESSAGE(STATUS "Added CVODE FCMIX module")
