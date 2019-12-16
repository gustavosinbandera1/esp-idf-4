idf_component_register(SRCS "WiFi.cpp" 
                            "WiFiEventHandler.cpp"
                    INCLUDE_DIRS "include"
                    )
set(EXTRA_COMPONENT_DIRS ${CMAKE_SOURCE_DIR}/components/Util)