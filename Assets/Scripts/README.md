# Project_HAL <img src="https://img.shields.io/badge/Unity-222222?style=for-the-badge&logo=unity&logoColor=white" align="absmiddle"/> <img src="https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=csharp&logoColor=white" align="absmiddle"/> <img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" align="absmiddle"/>

<p align="center">
  <img src="https://github.com/pwdab/Portfolio/blob/ver-3.0/images/Project_HAL/Project_HAL.png" alt="Project_HAL" width="75%">>
	<img src="https://github.com/pwdab/Portfolio/raw/ver-3.0/images/Project_HAL/features1.gif" width="32%">
	<img src="https://github.com/pwdab/Portfolio/raw/ver-3.0/images/Project_HAL/features2.gif" width="32%">
	<img src="https://github.com/pwdab/Portfolio/raw/ver-3.0/images/Project_HAL/features3.gif" width="32%">
</p>

## 🎮 게임 플레이   
- **직접 플레이**   
	<img src="https://upload.wikimedia.org/wikipedia/commons/1/12/Google_Drive_icon_%282020%29.svg" width="15" align="absmiddle"/> [구글 드라이브](https://drive.google.com/file/d/1hhAQobi0zfsc5SucmJjASzMnFuvB9EYg/view?usp=sharing)에서 Project_HAL_Demo.zip 파일을 다운로드 후 **HALNENG.exe**을 실행합니다.   
- **플레이 영상**      
	<img src="https://upload.wikimedia.org/wikipedia/commons/0/09/YouTube_full-color_icon_%282017%29.svg" width="15" align="absmiddle"/> [YouTube](https://youtu.be/5klQiKKPS54)에서 플레이 영상을 시청할 수 있습니다.

## 📌 프로젝트 소개
- **프로젝트 개요**   
  Unity로 제작한 탑뷰 2D 싱글플레이 어드벤처 게임\
  플레이어는 기본 공격과 카드 드로우 기반 스킬로 몬스터를 처치하고 스테이지를 클리어
- **개발 기간**   
  2024.03.09 ~ 2024.06.24
- **개발 상태**   
  데모 빌드 배포 완료 (개발 종료)
- **개발 환경**   
  Unity 2022.3.21f\
  Windows 10 (64bit)
- **멤버 구성**   
  프로그래밍 4명

## 🎯 담당 업무
- Sprite Atlas 기반 2D **애니메이션** 생성 및 Animation Controller **상태 전환** 구현   
- Collider2D 기반 **캐릭터-오브젝트 상호작용** 처리   
- 아이템 획득·폐기·이동이 가능한 **인벤토리 시스템** 구현

## ⚙️ 기능 구현
### 1. 캐릭터 이동 및 애니메이션 제어
<img src="images/implementation1.gif" width="33%">

- **기능 설명**:   
	- **RMB**: 클릭한 지점까지 이동   
	- **S**: 즉시 멈춤   
	- **LMB**: 공격   
	- **LMB + RMB**: 공격 도중 일정 시간 경과 후, 우클릭으로 공격 애니메이션 캔슬 가능
- **주요 기술**:
	- **Animation Controller**와 **Coroutine**을 활용한 애니메이션 제어
 	- **상태 변수** 기반 애니메이션 전환
- **구현 방법**:   
	- **이동**: FixedUpdate()에서 마우스 좌표를 받아 target_pos와 이동 벡터 계산 후 Rigidbody.velocity로 적용
		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
		
		```c#
		void FixedUpdate()
		{
		  ︙
			MoveCharacter_Mouse();
			Vectorreset();
			UpdateMovement();
		  ︙
		}
		
		private void MoveCharacter_Mouse()
		{
		  ︙
			// 움직일 수 있는 상황이면 target_pos 업데이트
			target_pos = GetMousePos();
		  ︙
		}
		
		public void Vectorreset()
		{
			// 이동 벡터 계산
			vector = target_pos - new Vector2(transform.position.x, transform.position.y);
		}
		
		private void UpdateMovement()
		{
		  ︙
			// 캐릭터를 vector와 velocity에 맞게 움직임
			rigidbody.velocity = vector * velocity;
		}
		```
		</details>

	- **멈춤**: CharacterStop() 호출 시 목표 좌표를 현재 위치로 고정 및 속도를 0으로 설정
		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
			
		```c#
		public void CharacterStop()
		{
			is_moveable = false;
			target_pos = transform.position;
			vector = Vector2.zero;
		}
		```	
		</details>
	
	- **애니메이션 재생**: PlayAnimation()에서 코루틴으로 공격/Idle 전환 제어
		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
		
		```c#
		public void PlayAnimation(string action)
		{
		  ︙
			if (animation_coroutine == null)
			{
				// 애니메이션 재생
				animation_coroutine = StartCoroutine(action, animator);
			}
		  ︙
		}
		```
		</details>

	- **애니메이션 캔슬**: 애니메이션이 일정 이상 재생되면 상태 변수를 true로 설정
		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
		
		```c#
		public IEnumerator Attack(Animator animator)
		{
		  ︙
			// 이전에 재생되던 애니메이션을 캔슬
			if (is_animation_cancelable)
			{
				while (!animator.GetCurrentAnimatorStateInfo(0).IsName("0_idle"))
				{
					animator.SetInteger(animationState, (int)AnimationStateEnum.idle);
					yield return null;
				}
				is_animation_cancelable = false;
			}
	
			// 현재 애니메이션을 재생
			while (!animator.GetCurrentAnimatorStateInfo(0).IsName("2_Attack_Bow"))
			{
				animator.SetInteger(animationState, (int)AnimationStateEnum.attack);
				yield return null;
			}
		  ︙
			while (animator.GetCurrentAnimatorStateInfo(0).normalizedTime < 1.0f)
			{
				is_animation_playing = true;
				// 현재 재생중인 애니메이션을 캔슬 가능
				if (animator.GetCurrentAnimatorStateInfo(0).normalizedTime > 0.5f)
				{
					is_animation_cancelable = true;
				}
				yield return null;
			}
		  ︙
		}
		```
		</details>

### 2. 오브젝트와 상호작용   
<img src="images/implementation2.gif" width="33%">

- **기능 설명**:   
	- 캐릭터는 맵에 배치된 오브젝트와 충돌해 데미지를 주거나 받고, 오브젝트를 파괴해 아이템을 획득
- **주요 기술**:
	- **Collider2D** 기반 충돌 감지 및 상호작용 처리  
	<p>
		<img src="images/implementation2-1.png" width="12%">
		<img src="images/implementation2-2.png" width="12%">
		<img src="images/implementation2-3.png" width="12%">
		<img src="images/implementation2-4.png" width="12%">
	</p>
  
- **구현 방법**:
	- **피격**: Enemy와 충돌 시, 1초 간격으로 데미지를 주는 코루틴 실행
		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
			
		```c#
		void OnCollisionEnter2D(Collision2D collision)
		{
			// 플레이어가 Enemy와 충돌
			if (collision.gameObject.CompareTag("Player"))
			{
				PlayerEntity player = collision.gameObject.GetComponent<PlayerEntity>();
				if (player.player_damage_coroutine == null)
				{
					// 일정 딜레이마다 데미지를 가함
				    player.player_damage_coroutine = StartCoroutine(player.DamageEntity(damage_scale, 1.0f, this.gameObject));
				}
			}
		}
		```
		</details>

	- **아이템 획득**: 아이템 오브젝트는 ItemType에 따라 처리
		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
			
		```c#
		private void OnTriggerEnter2D(Collider2D collision)
		{
  			// 플레이어가 아이템과 충돌
			if (is_alive && collision.gameObject.CompareTag(pickable_objects))
			{
  		 	 ︙
  				// ItemType에 따라 처리
				switch (hitObject.Item.ItemType)
				{
					case Item.ItemTypeEnum.GRASS:
					case Item.ItemTypeEnum.STONE:
					case Item.ItemTypeEnum.COIN:
  				 	 ︙
					case Item.ItemTypeEnum.HEALTH:
  		 			 ︙
				}
  		 	 ︙
			}
		}
		```
		</details>


### 3. 인벤토리 관리
<img src="images/implementation3.gif" width="33%">

- **기능 설명**:  
	- 플레이어는 아이템과 충돌해 습득, 드래그 앤 드롭으로 위치 변경 및 드랍

- **주요 기술**:  
	- 배열과 **Prefab**을 활용한 인벤토리 UI 및 아이템 관리

- **구현 방법**:
	- **슬롯 생성**: 슬롯 개수만큼 슬롯을 만들고 배열에 저장
		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
			
		```c#
		public void CreateSlots()
		{
		  ︙
			for (int i = 0; i < numSlots; i++)
			{
  				// 슬롯 생성
				GameObject newSlot = Instantiate(slotPrefab);
			  ︙
  				// 배열 생성한 슬롯 저장
				slots[i] = newSlot;
			}
		  ︙
		}
		```
  		</details>

	- **아이템 추가**: 동일 아이템이면 수량 합산, 아니면 빈 슬롯에 추가  
 		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
			
		```c#
		public bool AddItem(PickableObjects itemToAdd)
		{
		  ︙
			// 인벤토리에 존재하면 수량을 더함
			int qty = int.Parse(qtyText.text);
			qty += itemToAdd.Quantity;
			qtyText.text = qty.ToString();
		  ︙
			// 인벤토리에 존재하지 않으면 아이템을 생성
			items[i] = Instantiate(itemToAdd.Item);
		  ︙
		}
		```
  		</details>

	- **아이템 이동**: 빈 슬롯이면 단순 이동, 아니면 슬롯 간 교체  
  		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
			
		```c#
		public bool MoveItem(int targetSlotNum)
		{
  		  ︙
  			// 목표 슬롯에 아이템 생성
			if (AddItemAt(new PickableObjects(items[clicked_slot], quantity), targetSlotNum))
  			{
  			  ︙
  				// 기존 슬롯의 아이템 삭제
				return DeleteItem(clicked_slot, quantity);
  			}
			else
  			{
  			  ︙
  				// 두 슬롯을 교체
  				AddItemAt(srcTmp, targetSlotNum);
				AddItemAt(dstTmp, clicked_slot);
  			  ︙
			}
		  ︙
		}
		```
  		</details>
	
	- **아이템 드랍**: Prefab 로드 후 월드에 스폰, 슬롯 초기화  
  		<details>
			<summary> <em>코드 펼치기/접기</em></summary>
			
		```c#
		private bool DropItem()
		{
		  ︙
  			// 아이템 스폰
			prefab_to_spawn = Resources.Load<GameObject>(prefab_path);
	  
			// 슬롯 초기화
	        ClearSlot(clicked_slot);
	        ClearSlot(numSlots);
  		  ︙
		}
		```
  		</details>

## 🛠 이슈 및 해결 과정 
### 캐릭터 애니메이션 전환 오류
- **문제**   
  간헐적으로 캐릭터의 애니메이션(ex. 공격)이 재생되지 않는 현상 발생   
  <div>
	<img src="images/issues1.gif" width="25%">   
  </div>
  
- **원인**   
  루프로 설정된 Idle 애니메이션의 진행도(normalizedTime) 값이 1.0f를 넘어 누적\
  Animator의 상태가 실제로 전환되기도 전에 애니메이션 종료 조건이 먼저 충족되어 다음 루틴이 실행되지 않음
	<details>
		<summary> <em>코드 펼치기/접기</em></summary>
		
	```C#
	// Before
	public IEnumerator Attack(Animator animator)
	{
	  ︙
		// 캐릭터의 상태를 "Attack"으로 바꿈
		if (!animator.GetCurrentAnimatorStateInfo(0).IsName("2_Attack_Bow"))
		{
			animator.SetInteger(animationState, (int)AnimationStateEnum.attack);
		}
	
		// state가 아직 "Attack"으로 바뀌지 않았음에도 이전에 재생 중이던 Idle의 normalizedTime이 1.0f을 넘으면 아래의 코드가 작동하지 않음
		while (animator.GetCurrentAnimatorStateInfo(0).normalizedTime < 1.0f)
		{
			// 현재 애니메이션이 종료될 때까지 기다림
		}
	  ︙
	}
	```
	</details>
 
- **해결**   
  Animator의 상태가 확실히 바뀌도록 대기하는 로직을 추가
	<details>
		<summary> <em>코드 펼치기/접기</em></summary>
		
	```C#
	// After
	public IEnumerator Attack(Animator animator)
	{
	  ︙
		// 캐릭터의 상태를 확실히 "Attack"으로 바꿀 때까지 기다림
		while (!animator.GetCurrentAnimatorStateInfo(0).IsName("2_Attack_Bow"))
		{
			animator.SetInteger(animationState, (int)AnimationStateEnum.attack);
			yield return null;
		}
	
		// state가 확실히 "Attack"으로 바뀌었으므로 normalizedTime은 0.0f부터 시작
		// 따라서 아래의 코드가 정상적으로 작동함
		while (animator.GetCurrentAnimatorStateInfo(0).normalizedTime < 1.0f)
		{
			// 현재 애니메이션이 종료될 때까지 기다림
		}
	  ︙
	}
	```
	</details>
 
- **결과**   
  애니메이션이 정상적으로 재생되지 않던 문제 해결\
  추후 애니메이션의 후딜레이를 캔슬하는 기능으로 확장해 조작감과 전투 흐름을 개선   
  <div>
	<img src="images/issues2.gif" width="25%">   
  </div>


## 프로젝트 구조
```plaintext
Assets/
├── Resources/
│   ├── Prefabs/
│   │   ├── UI/
│   │   │   ├── HPBar/
│   │   │   │   ├── BossHPBar.prefab                # 보스 몬스터의 HP Bar 프리팹
│   │   │   │   ├── EnemyHPBarUI.prefab             # 일반 몬스터의 HP Bar 프리팹
│   │   │   └── └── PlayerHPBarUI.prefab            # 플레이어의 HP Bar 프리팹
│   │   │   ├── Inventory/
│   │   │   │   ├── InventoryUI.prefab              # 인벤토리 UI 프리팹
│   │   └── └── └── SlotUI.prefab                   # 인벤토리 슬롯 UI 프리팹
│   │   ├── CoinObject.prefab                       # 맵에 드랍할 수 있는 Coin 프리팹
│   │   ├── Dummy.prefab                            # 더미(일반 몬스터) 프리팹
│   │   ├── HeartObject.prefab                      # 맵에 드랍할 수 있는 Heart 프리팹
│   │   ├── GameManager.prefab                      # GameManager 프리팹
│   └── └── PlayerObject.prefab                     # 플레이어 객체 프리팹
│   ├── ScriptableObjects/
│   │   ├── Coin.asset                              # Item을 상속받아 구현한 Coin Item
│   │   ├── DummyHPManager.asset                    # StatManager를 상속받아 Dummy의 Stat을 구현한 StatManager
│   │   ├── Heart.asset                             # Item을 상속받아 구현한 Heart Item
│   │   ├── Item.cs                                 # Item을 정의한 ScriptableObject
│   │   ├── StatManager.asset                       # StatManager를 상속받아 플레이어의 Stat을 구현한 StatManager
│   └── └── StatManager.cs                          # Entity Stat을 정의한 ScriptableObject
├── Scripts/
│   ├── Entity/
│   │   ├── Entity.cs                               # Entity의 최상위 클래스
│   │   ├── EnemyEntity.cs                          # Entity를 상속받아 적 몬스터의 초기화, 피격 및 공격 등에 관련한 코드
│   └── └── PlayerEntity.cs                         # Entity를 상속받아 플레이어의 초기화, 이동, 애니메이션 재생 등에 관련한 코드
│   ├── Manager/
│   │   ├── GameManager.cs                          # 게임 매니저 관련 코드 (싱글톤 패턴)
│   └── └── VirtualCameraManager.cs                 # 가상 카메라와 관련 변수와 함수들에 관련한 코드
│   ├── UI/
│   │   ├── HPBarUI.cs                              # 플레이어의 화면에 표시되는 HP UI
│   │   ├── InventorySlotUI.cs                      # 인벤토리 슬롯 하나에 해당하는 UI
│   └── └── InventoryUI.cs                          # InventorySlotUI 여러개를 모은 하나의 Inventory UI. Inventory 상 아이템의 습득, 폐기, 이동에 관련한 코드
│   └── PickableObjects.cs                          # 필드에 드랍할 수 있는 Item에 관련한 코드
├── SPUM/
│   ├── Res/
│   │   ├── Animation/
│   │   │   ├── Clip/
│   │   │   └── └── *.anim                           # 캐릭터의 애니메이션 파일들
└── └── └── └── AnimationNewController.controller    # 캐릭터의 애니메이션을 상태에 따라 제어

```
