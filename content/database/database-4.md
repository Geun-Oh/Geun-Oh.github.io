+++
title = 'Database 3'
date = 2024-09-07T18:59:35+09:00
draft = false
+++

### /mnt/data와 /mnt/log 가 다른 디스크에 마운트됨을 확인하기

단순히 경로를 표시하는 것만으로는 확실하게 물리적 격리가 이루어지고 있는지 확인할 수 없다.
각 마운트 지점이 물리적으로 서로 다른 디스크에 매핑되어 있는지 확인하려면, 다음과 같은 명령어를 사용한다.

#### 1. `df`

`df` 는 파일 시스템의 디스크 사용량을 확인하는 데 사용되며, 각 마운트 포인트가 어떤 디스크에 연결되어 있는지 확인 가능하다.

```bash
df -h /mnt/data /mnt/log
```

해당 명령어 실행 시 아래와 같이 `/mnt/data`와 `/mnt/log` 가 각각 어떤 디스크와 디바이스에 마운트되어 있는지 확인할 수 있다.

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G   20G   80G  20% /mnt/data
/dev/sdb1       200G   50G  150G  25% /mnt/log
```

위 결과에서 `/mnt/data` 는 `/dev/sda1`에, `/mnt/log` 는 `/dev/sdb1`에 마운트되어 있음을 확인할 수 있었다.
이렇게 다른 디스크에 마운트되어 있는 경우 물리적으로 다른 디스크에 저장된다는 것이 확인 가능하다.

#### 2. `lsblk`

`lsblk`는 블록 디바이스의 계층 구조를 보여주며, 마운트 지점과 디바이스 간의 매핑을 확인하는 데 유용하다.

```bash
lsblk
```

위 명령어를 실행하면 모든 블록 디바이스 및 그에 연결된 마운트 지점의 목록을 볼 수 있다. 아래 그 실행 예시이다.

```bash
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0   100G  0 disk
└─sda1   8:1    0   100G  0 part /mnt/data
sdb      8:16   0   200G  0 disk
└─sdb1   8:17   0   200G  0 part /mnt/log
```

여기서 `sda`와 `sdb`는 각각 다른 물리적 디스크를 나타내며, `/mnt/data` 와 `/mnt/log` 가 각각 다른 디스크에 마운트되어 있는 것을 확인할 수 있다.

#### 3. `mount`

`mount` 를 사용하면 현재 시스템에서 마운트된 모든 파일 시스템 확인이 가능하다.

```bash
mount | grep -E "/mnt/data|/mnt/log"
```

이 명령은 `/mnt/data` 와 `/mnt/log` 가 어떤 디바이스에 마운트되어 있는지 보여준다. 아래는 그 실행 예시.

```bash
/dev/sda1 on /mnt/data type ext4 (rw,relatime)
/dev/sdb1 on /mnt/log type ext4 (rw,relatime)
```

이 출력에서 `/mnt/data`는 `/dev/sda1`에, `/mnt/log`는 `/dev/sdb1`에 마운트되어 있음을 확인할 수 있다.

#### 4. `/etc/fstab` 확인

시스템 재부팅 시 자동으로 마운트되는 파일 시스템을 정의하는 `/etc/fstab` 파일을 확인하는 것도 좋다.
다음 명령어를 사용한다.

```bash
cat /etc/fstab
```

`/etc/fstab` 파일 내용 중 `/mnt/data`와 `/mnt/log`가 서로 다른 디스크 디바이스에 설정되어 있는지 확인 가능하다.

```bash
/dev/sda1   /mnt/data  ext4  defaults  0  2
/dev/sdb1   /mnt/log   ext4  defaults  0  2
```

여기서도 `/mnt/data`와 `/mnt/log`가 서로 다른 디스크에 설정되어 있는 것을 확인 가능하다.

---

이러한 방법들을 통해 `/mnt/data`와 `/mnt/log`가 물리적으로 격리되어 있는지 여부를 확인할 수 있다.