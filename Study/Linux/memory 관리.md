### linux cache 종류
1. buffer cache
   - 블럭 디바이스 드라이버들이 사용하는 데이터 버퍼들을 갖는다.(모든 하드디스크는 블럭 장치이다.)
   - 데이터가 버퍼 캐시에서 발견되면 하드디스크같은 물리적 장치에 접근할 필요가 없다. 
2. page cache
   - 디스크상의 이미지와 데이터에 접근하는 속도를 높이기 위해 사용한다.
   - 파일의 논리적 내용을 페이지 단위로 캐시하기 위해 사용하고, 파일과 파일 내의 오프셋을 통해 접근한다.
   - 디스크에서 메모리로 페이지들을 읽어들이면 페이지들은 페이지 캐시에 캐시된다.
3. swap cache
   - 더티 페이지들을 스왑 파일에 저장하는데 저장된 후 변경되지 않았다면, 그 페이지가 스왑 아웃 될 때는 이미 해당 내용이 스왑 파일에 있으므로, 스왑 파일에 등록할 필요가 없다.
   - 스와핑이 심하게 일어나는 시스템에서 유용하다
4. hardware cache
   - 페이지 테이블 엔트리(PTE)의 캐시를 말한다. (= TLB)

#### 리눅스 페이지 캐시
![[스크린샷 2024-06-24 오후 11.13.48.png]]
- 리눅스에서는 메모리 매핑된 파일이 읽힐 때 한 페이지씩 읽혀지고, 이들 페이지는 페이지 캐시에 저장된다.
- 페이지 캐시는`mem_map_t` 자료구조에 대한 포인터들의 벡터인 `page_hash_table`로 구성되어 있다.
- 리눅스 각 파일은 VFS inode에 의해 식별되며 1:1로 매핑된다.
- 페이지가 캐시에 있으면, `mem_map_t` 자료구조에 대한 포인터가 페이지 폴트 처리 코드로 되돌려진다. 

### 페이지 스왑아웃 & 폐기(discarding)
- 물리적 메모리가 부족하면 리눅스 메모리 관리 서브시스템은 물리적 메모리를 해제하려고 한다. 이 일은 커널데몬스왑(`kswapd`)이 맡는다.
- 커널데몬스왑(`kswapd`)은 `init process`에 의해 시작되고, 커널 스왑 타이머가 주기적으로 만료될 때마다 free page 수를 확인한다.
- `free_pages_high`, `free_pages_low` 라는 두 개의 변수를 사용하며 프리 페이지 수가`free_pages_high`보다 크다면 아무 일도 하지 않는다. 
- `free_pages_low` 이하로 떨어지게 되면 커널데몬스왑은 물리적 페이지의 수를 줄이기 위해서 다음 세가지 방법을 시도한다.
	1. 버퍼 캐시와 페이지 캐시 크기 줄이기
	2. 시스템 V 공유 메모리 페이지 스왑 아웃
	3. 페이지를 스왑 아웃하고 폐기

# References
- https://www.tencentcloud.com/ko/document/product/213/40504
- https://jujupapa.tistory.com/31
- http://www.secretmango.com/jimb/Whitepapers/slabs/slab.html
- https://jeongzero.oopy.io/132fed8f-5cfd-4f43-990c-61584744b4d0
- https://wiki.kldp.org/Translations/html/The_Linux_Kernel-KLDP/tlk3.html