[ Client A ] --(WS)--+  
[ Client B ] --(WS)--+--> [ Chat Server 1 ] --+  
[ Client C ] --(WS)--+                        |  
                                             v
                                       [ Routing Service ]
                                             |
                                             v
                                     [ Session Storage ]
                              (client_id -> server_id mapping)

                                             |
                                             v
                                     [ Message Queues ]
                                   (one queue per server)
                                             |
                  +--------------------------+-------------------------+
                  |                                                    |
                  v                                                    v
          [ Chat Server 1 ] <--- subscribes queue_1            [ Chat Server 2 ] <--- subscribes queue_2
          - Maintains WS connections                           - Maintains WS connections
          - Emits to correct client                            - Emits to correct client
                  |                                                    |
                  v                                                    v
           [ Client A, B ]                                      [ Client C, D ]