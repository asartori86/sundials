# ---------------------------------------------------------------
# Programmer(s): Daniel R. Reynolds @ SMU
#                David J. Gardner @ LLNL
# ---------------------------------------------------------------
# LLNS/SMU Copyright Start
# Copyright (c) 2015, Southern Methodist University and
# Lawrence Livermore National Security
#
# This work was performed under the auspices of the U.S. Department
# of Energy by Southern Methodist University and Lawrence Livermore
# National Laboratory under Contract DE-AC52-07NA27344.
# Produced at Southern Methodist University and the Lawrence
# Livermore National Laboratory.
#
# All rights reserved.
# For details, see the LICENSE file.
# LLNS/SMU Copyright End
# Copyright (c) 2013, Southern Methodist University.
# All rights reserved.
# For details, see the LICENSE file.
# ---------------------------------------------------------------
# CMakeLists.txt file for the FARKODE F90 serial examples
# ---------------------------------------------------------------

# Example lists are tuples "name\;type" where the type is
# 'develop' for examples excluded from 'make test' in releases

# Examples using SUNDIALS linear solvers
SET(FARKODE_examples
  "ark_bruss\;develop"
  )

# Examples using LAPACK linear solvers
SET(FARKODE_examples_BL
  )

# Add sparse solvers examples for 64-bit indextype configurations only,
# not compatible with 32-bit indextype
IF(SUNDIALS_INDEX_SIZE MATCHES "64")

  # Examples using KLU linear solver
  SET(FARKODE_examples_KLU
    "ark_bruss1D_FEM_klu\;develop"
    )

  # Examples using SuperLU_MT linear solver
  SET(FARKODE_examples_SUPERLUMT
    )

ENDIF()

# Auxiliary files to install
SET(ARKODE_extras
  plot_sol.py
  plot_brusselator1D_FEM.py
  )

# Specify libraries to link against (through the target that was used to
# generate them) based on the value of the variable LINK_LIBRARY_TYPE
IF(LINK_LIBRARY_TYPE MATCHES "static")
  SET(ARKODE_LIB sundials_arkode_static)
  SET(NVECS_LIB sundials_nvecserial_static)
  SET(FNVECS_LIB sundials_fnvecserial_static)
ELSE()
  SET(ARKODE_LIB sundials_arkode_shared)
  SET(NVECS_LIB sundials_nvecserial_shared)
  SET(FNVECS_LIB sundials_fnvecserial_shared)
ENDIF()

# Only static FCMIX libraries are available
SET(FARKODE_LIB sundials_farkode_static)

# Set-up linker flags and link libraries
SET(SUNDIALS_LIBS ${FARKODE_LIB} ${ARKODE_LIB} ${FNVECS_LIB} ${NVECS_LIB} ${EXTRA_LINK_LIBS})


# Add the build and install targets for each example
FOREACH(example_tuple ${FARKODE_examples})

  # parse the example tuple
  LIST(GET example_tuple 0 example)
  LIST(GET example_tuple 1 example_type)

  # example source files
  ADD_EXECUTABLE(${example} ${example}.f90)

  SET_TARGET_PROPERTIES(${example} PROPERTIES FOLDER "Examples")

  # add example to regression tests
  SUNDIALS_ADD_TEST(${example} ${example}
    ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    ANSWER_FILE ${example}.out
    EXAMPLE_TYPE ${example_type})

  # libraries to link against
  TARGET_LINK_LIBRARIES(${example} ${SUNDIALS_LIBS})

   # install example source and out files
  IF(EXAMPLES_INSTALL)
    INSTALL(FILES ${example}.f90 ${example}.out
      DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial)
  ENDIF()

ENDFOREACH(example_tuple ${FARKODE_examples})


# Add the build and install targets for each LAPACK example (if needed)
IF(LAPACK_FOUND)

  # Sundials LAPACK linear solver modules
  IF(LINK_LIBRARY_TYPE MATCHES "static")
    SET(SUNLINSOLLAPACK_LIBS
      sundials_sunlinsollapackband_static
      sundials_sunlinsollapackdense_static
      sundials_fsunlinsollapackband_static
      sundials_fsunlinsollapackdense_static)
  ELSE()
    SET(SUNLINSOLLAPACK_LIBS
      sundials_sunlinsollapackband_shared
      sundials_sunlinsollapackdense_shared
      sundials_fsunlinsollapackband_shared
      sundials_fsunlinsollapackdense_shared)
  ENDIF()

  # LAPACK libraries
  LIST(APPEND SUNLINSOLLAPACK_LIBS ${LAPACK_LIBRARIES})

  # BLAS libraries
  IF(BLAS_FOUND)
    LIST(APPEND SUNLINSOLLAPACK_LIBS ${BLAS_LIBRARIES})
  ENDIF(BLAS_FOUND)

  FOREACH(example_tuple ${FARKODE_examples_BL})

    # parse the example tuple
    LIST(GET example_tuple 0 example)
    LIST(GET example_tuple 1 example_type)

    # example source files
    ADD_EXECUTABLE(${example} ${example}.f90)

    SET_TARGET_PROPERTIES(${example} PROPERTIES FOLDER "Examples")

    # add example to regression tests
    SUNDIALS_ADD_TEST(${example} ${example}
      ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
      ANSWER_FILE ${example}.out
      EXAMPLE_TYPE ${example_type})

    # libraries to link against
    TARGET_LINK_LIBRARIES(${example} ${SUNDIALS_LIBS} ${SUNLINSOLLAPACK_LIBS})

     # install example source and out files
    IF(EXAMPLES_INSTALL)
      INSTALL(FILES ${example}.f90 ${example}.out
        DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial)
    ENDIF()

  ENDFOREACH(example_tuple ${FARKODE_examples_BL})

ENDIF(LAPACK_FOUND)


# Add the build and install targets for each KLU example (if needed)
IF(KLU_FOUND)

  # Sundials KLU linear solver module
  IF(LINK_LIBRARY_TYPE MATCHES "static")
    SET(SUNLINSOLKLU_LIBS
      sundials_sunlinsolklu_static
      sundials_fsunlinsolklu_static)
  ELSE()
    SET(SUNLINSOLKLU_LIBS
      sundials_sunlinsolklu_shared
      sundials_fsunlinsolklu_shared)
  ENDIF()

  # KLU libraries
  LIST(APPEND SUNLINSOLKLU_LIBS ${KLU_LIBRARIES})

  FOREACH(example_tuple ${FARKODE_examples_KLU})

    # parse the example tuple
    LIST(GET example_tuple 0 example)
    LIST(GET example_tuple 1 example_type)

    # example source files
    ADD_EXECUTABLE(${example} ${example}.f90)

    SET_TARGET_PROPERTIES(${example} PROPERTIES FOLDER "Examples")

    # add example to regression tests
    SUNDIALS_ADD_TEST(${example} ${example}
      ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
      ANSWER_FILE ${example}.out
      EXAMPLE_TYPE ${example_type})

    # libraries to link against
    TARGET_LINK_LIBRARIES(${example} ${SUNDIALS_LIBS} ${SUNLINSOLKLU_LIBS})

     # install example source and out files
    IF(EXAMPLES_INSTALL)
      INSTALL(FILES ${example}.f90 ${example}.out
        DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial)
    ENDIF()

  ENDFOREACH(example_tuple ${FARKODE_examples_KLU})

ENDIF(KLU_FOUND)


# Add the build and install targets for each SuperLU_MT example (if needed)
IF(SUPERLUMT_FOUND)

  # Sundials SuperLU_MT linear solver module
  IF(LINK_LIBRARY_TYPE MATCHES "static")
    SET(SUNLINSOLSLUMT_LIBS
      sundials_sunlinsolsuperlumt_static
      sundials_fsunlinsolsuperlumt_static)
  ELSE()
    SET(SUNLINSOLSLUMT_LIBS
      sundials_sunlinsolsuperlumt_shared
      sundials_fsunlinsolsuperlumt_shared)
  ENDIF()

  # SuperLU_MT libraries
  LIST(APPEND SUNLINSOLSLUMT_LIBS ${SUPERLUMT_LIBRARIES})

  # BLAS libraries
  IF(BLAS_FOUND)
    LIST(APPEND SUNLINSOLSLUMT_LIBS ${BLAS_LIBRARIES})
  ENDIF(BLAS_FOUND)

  FOREACH(example_tuple ${FARKODE_examples_SUPERLUMT})

    # parse the example tuple
    LIST(GET example_tuple 0 example)
    LIST(GET example_tuple 1 example_type)

    # example source files
    ADD_EXECUTABLE(${example} ${example}.f90)

    SET_TARGET_PROPERTIES(${example} PROPERTIES FOLDER "Examples")

    # add example to regression tests
    SUNDIALS_ADD_TEST(${example} ${example}
      ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
      ANSWER_FILE ${example}.out
      EXAMPLE_TYPE ${example_type})

    # libraries to link against
    TARGET_LINK_LIBRARIES(${example} ${SUNDIALS_LIBS} ${SUNLINSOLSLUMT_LIBS})

     # install example source and out files
    IF(EXAMPLES_INSTALL)
      INSTALL(FILES ${example}.f90 ${example}.out
        DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial)
    ENDIF()

  ENDFOREACH(example_tuple ${FARKODE_examples_SUPERLUMT})

ENDIF(SUPERLUMT_FOUND)


# create Makfile and CMakeLists.txt for examples
IF(EXAMPLES_INSTALL)

  # Install the README file
  INSTALL(FILES README DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial)

  # Install the extra files
  FOREACH(extrafile ${ARKODE_extras})
    INSTALL(FILES ${extrafile} DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial)
  ENDFOREACH()

  # Prepare substitution variables for Makefile and/or CMakeLists templates
  SET(SOLVER "ARKODE")
  SET(SOLVER_LIB "sundials_arkode")
  SET(SOLVER_FLIB "sundials_farkode")

  EXAMPLES2STRING(FARKODE_examples EXAMPLES)

  IF(LAPACK_FOUND)
    EXAMPLES2STRING(FARKODE_examples_BL EXAMPLES_BL)
  ELSE()
    SET(EXAMPLES_BL "")
  ENDIF()

  IF(KLU_FOUND)
    EXAMPLES2STRING(FARKODE_examples_KLU EXAMPLES_KLU)
  ELSE()
    SET(EXAMPLES_KLU "")
  ENDIF()

  IF(SUPERLUMT_FOUND)
    EXAMPLES2STRING(FARKODE_examples_SUPERLUMT EXAMPLES_SLUMT)
  ELSE()
    SET(EXAMPLES_SLUMT "")
  ENDIF()

  # Regardless of the platform we're on, we will generate and install
  # CMakeLists.txt file for building the examples. This file  can then
  # be used as a template for the user's own programs.

  # generate CMakelists.txt in the binary directory
  CONFIGURE_FILE(
    ${PROJECT_SOURCE_DIR}/examples/templates/cmakelists_serial_F90_ex.in
    ${PROJECT_BINARY_DIR}/examples/arkode/F90_serial/CMakeLists.txt
    @ONLY
    )

  # install CMakelists.txt
  INSTALL(
    FILES ${PROJECT_BINARY_DIR}/examples/arkode/F90_serial/CMakeLists.txt
    DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial
    )

  # On UNIX-type platforms, we also  generate and install a makefile for
  # building the examples. This makefile can then be used as a template
  # for the user's own programs.

  IF(UNIX)
    # generate Makefile and place it in the binary dir
    CONFIGURE_FILE(
      ${PROJECT_SOURCE_DIR}/examples/templates/makefile_serial_F90_ex.in
      ${PROJECT_BINARY_DIR}/examples/arkode/F90_serial/Makefile_ex
      @ONLY
      )
    # install the configured Makefile_ex as Makefile
    INSTALL(
      FILES ${PROJECT_BINARY_DIR}/examples/arkode/F90_serial/Makefile_ex
      DESTINATION ${EXAMPLES_INSTALL_PATH}/arkode/F90_serial
      RENAME Makefile
      )
  ENDIF(UNIX)

ENDIF(EXAMPLES_INSTALL)
