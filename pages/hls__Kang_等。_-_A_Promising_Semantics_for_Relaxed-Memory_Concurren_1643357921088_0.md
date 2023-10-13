file-path:: ../assets/Kang_等。_-_A_Promising_Semantics_for_Relaxed-Memory_Concurren_1643357921088_0.pdf

- promises:a thread may promise to execute a write in the future, thus enablingother threads to read from that write out of order
  ls-type:: annotation
  hl-page:: 1
  id:: 61f3a843-4622-487b-a8c4-9a59ec45cdbb
- sequentially consistent
  ls-type:: annotation
  hl-page:: 1
  id:: 61ffc544-21ad-4e8a-aca6-4d488e85e19d
- one must therefore insert expensive fence instructionsto subvert the efforts of the hardware
  ls-type:: annotation
  hl-page:: 1
  id:: 6200cc38-0f2a-4394-af9e-d785a344144d
- constant propagation
  ls-type:: annotation
  hl-page:: 1
  id:: 6200d19f-9a17-4c98-a53d-22079291e01b
- DRF
  ls-type:: annotation
  hl-page:: 1
  id:: 6200e38f-c253-4e52-923d-92fe1167fb02
- high-level reasoning
  ls-type:: annotation
  hl-page:: 1
  id:: 6200e393-2728-468f-a2f8-3ab9219f9316
- brittle 
  ls-type:: annotation
  hl-page:: 2
  id:: 62011eb2-0d31-406d-8002-4c0a258810c5
- The C++ model formalizes valid executions as graphs of memoryaccess events (think: partially-ordered traces) subject to a set ofcoherence axioms, and the same coherent event graph that describesa valid execution of LB in whichareceives1also describes a validexecution of LBd in whichareceives1.
  ls-type:: annotation
  hl-page:: 2
  id:: 62011fb0-1e09-4a5f-85cd-d35c1b7d19cc
- operational semantics,a threadTmay nondeterministically “promise” to write a valuevto a memory locationxat some point in the future.
  ls-type:: annotation
  hl-page:: 2
  id:: 62012608-a2eb-4426-b613-209b3c8e6ce7
- thread-locally certify
  ls-type:: annotation
  hl-page:: 2
  id:: 620130a0-1950-4091-bb89-27cee34df15c
- con-sume reads
  ls-type:: annotation
  hl-page:: 2
  id:: 62029d53-30cd-44fc-b485-2d2379f453da
- per-location coherence
  ls-type:: annotation
  hl-page:: 3
  id:: 6203ddd3-b9c6-4f36-9dd7-f815eb70ea6b
- eclarative semantics 
  ls-type:: annotation
  hl-page:: 3
  id:: 6203e396-4cd8-46c5-96f6-30ca532869bc
- d
  ls-type:: annotation
  hl-page:: 3
  id:: 6203e3a5-862f-47eb-9ae6-2b43c5bf7b94
- eventgraphs
  ls-type:: annotation
  hl-page:: 3
  id:: 6203e3c3-8071-4f49-819b-9ea3aedc33b3
- (unique)
  ls-type:: annotation
  hl-page:: 3
  id:: 62093785-dfc2-4b2c-a54e-75d03ca5b288
- unused
  ls-type:: annotation
  hl-page:: 3
  id:: 6209397a-3d08-4901-9211-e18d8db4c406
- t just falls out from our rules concerning timestamps.In particular, ifTwere to promise〈x:v@t〉, and then were to readfrom its own promise, thenT’s view ofxwould be updated tot,and there would be no way forTto subsequently fulfill the promisebecause it would have to pick a timestamp strictly greater thantwhen performing the assignmentx:=v.
  ls-type:: annotation
  hl-page:: 4
  id:: 620a0705-6c0d-43a9-9e4d-a95ed1fb63e5
- is actuallyallowed (though not yet observed) by the ARM memory model
  ls-type:: annotation
  hl-page:: 4
  id:: 620a6452-2585-4e70-84e1-10c7b5382a5d