if(BUILD_MONGO_DB_PLUGIN)
  file(GLOB HEADERS "include/eosio/mongo_db_plugin/*.hpp")
  add_library( mongo_db_plugin
               mongo_db_plugin.cpp
               ${HEADERS} )

  find_package(libmongoc-1.0 1.8)

  if (libmongoc-1.0_FOUND)

      # EOS has no direct dependencies on libmongoc but its shared libraries
      # will need to be present at runtime for the C++ libraries we use:
      # libbsoncxx & libmongocxx (both from github.com/mongodb/mongo-cxx-driver)

      # The *.cmake package files provided by mongo-cxx-driver don't give us the
      # absolute path to the libraries, which is needed whenever they are not
      # installed in system-known locations.  CMake requires the absolute paths
      # in target_link_libraries() since we are builiding an archive and the
      # link step for all executables using this archive must include the
      # mongo-cxx-driver libraries libmongocxx and libbsoncxx.

      find_package(libbsoncxx REQUIRED)
      message(STATUS "Found bsoncxx headers: ${LIBBSONCXX_INCLUDE_DIRS}")
      find_library(EOS_LIBBSONCXX ${LIBBSONCXX_LIBRARIES}
                   PATHS ${LIBBSONCXX_LIBRARY_DIRS} NO_DEFAULT_PATH)
      message(STATUS "Found bsoncxx library: ${EOS_LIBBSONCXX}")

      find_package(libmongocxx REQUIRED)
      message(STATUS "Found mongocxx headers: ${LIBMONGOCXX_INCLUDE_DIRS}")
      find_library(EOS_LIBMONGOCXX ${LIBMONGOCXX_LIBRARIES}
                   PATHS ${LIBMONGOCXX_LIBRARY_DIRS} NO_DEFAULT_PATH)
      message(STATUS "Found mongocxx library: ${EOS_LIBMONGOCXX}")
  else()
      message("Could NOT find MongoDB. mongo_db_plugin with MongoDB support will not be included.")
      # sudo apt-get install pkg-config libssl-dev libsasl2-dev
      # wget https://github.com/mongodb/mongo-c-driver/releases/download/1.8.0/mongo-c-driver-1.8.0.tar.gz
      # tar xzf mongo-c-driver-1.8.0.tar.gz
      # cd mongo-c-driver-1.8.0
      # ./configure --disable-automatic-init-and-cleanup --enable-static
      # make
      # sudo make install
      #
      # git clone https://github.com/mongodb/mongo-cxx-driver.git --branch releases/stable --depth 1
      # cd mongo-cxx-driver/build
      # cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local ..
      # sudo make EP_mnmlstc_core
      # make
      # sudo make install
      #
      # sudo apt-get install mongodb
  endif()

  target_include_directories(mongo_db_plugin
          PRIVATE ${LIBMONGOCXX_INCLUDE_DIRS} ${LIBBSONCXX_INCLUDE_DIRS}
          PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include"
          )

  target_compile_definitions(mongo_db_plugin
    PRIVATE ${LIBMONGOCXX_DEFINITIONS} ${LIBBSONCXX_DEFINITIONS}
    )

  target_link_libraries(mongo_db_plugin
          PUBLIC chain_plugin eosio_chain appbase
          ${EOS_LIBMONGOCXX} ${EOS_LIBBSONCXX}
          )
else()
  message("mongo_db_plugin not selected and will be omitted.")
endif()
