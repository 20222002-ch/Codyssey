# 처음부터 재현하기

아래는 재현용 명령이다. Docker Desktop을 실행하고 Linux 엔진이 준비된 **PowerShell**에서 진행한다. 기존 실습과 구분하려고 새 이름 및 8082·8083 포트를 사용한다. 해당 포트도 사용 중이면 번호를 바꾼다.

## 1. 복제·빌드·실행

```powershell
git clone https://github.com/20222002-ch/Codyssey.git
cd Codyssey
docker --version
docker info
docker build -t codyssey-repro:1.0 .
docker run -d --name codyssey-repro-web -p 8082:80 codyssey-repro:1.0
docker run -d --name codyssey-repro-bind -p 8083:80 --mount "type=bind,source=$($PWD.Path)\site,target=/usr/share/nginx/html,readonly" codyssey-repro:1.0
curl.exe http://localhost:8082
curl.exe http://localhost:8083
```

두 포트의 초기 응답은 모두 현재 소스와 같아야 한다. 브라우저에서 http://localhost:8082 및 http://localhost:8083을 열어 확인한다.

## 2. 바인드 마운트 변경 확인

```powershell
$originalHtml = Get-Content -Raw -Encoding utf8 .\site\index.html
$originalHtml.Replace('바인드 마운트 변경 반영 성공!', '재현 확인: 바인드 마운트') | Set-Content -Encoding utf8 .\site\index.html
curl.exe http://localhost:8082
curl.exe http://localhost:8083
```

8082는 빌드 시 제목, 8083만 재현 확인: 바인드 마운트로 표시되는지 확인한다. 주소창을 포함해 전후를 캡처한 뒤 원래 내용을 복원한다.

```powershell
$originalHtml | Set-Content -Encoding utf8 .\site\index.html
```

## 3. 볼륨 삭제 전후 검증

```powershell
docker volume create codyssey-repro-data
docker run -d --name codyssey-repro-before --mount type=volume,source=codyssey-repro-data,target=/data ubuntu sleep infinity
docker exec codyssey-repro-before bash -c "echo 'Codyssey volume persistence test' > /data/hello.txt"
docker exec codyssey-repro-before cat /data/hello.txt
docker rm -f codyssey-repro-before
docker run -d --name codyssey-repro-after --mount type=volume,source=codyssey-repro-data,target=/data ubuntu sleep infinity
docker exec codyssey-repro-after cat /data/hello.txt
```

처음과 마지막 cat 출력이 같은지 확인하고 명령·출력을 연속으로 저장한다.

## 4. attach 연결 해제 비교

```powershell
docker run -it --name codyssey-repro-attach ubuntu bash
```

Bash에서 echo "attach verification"을 실행하고 Ctrl+P, Ctrl+Q로 분리한다. PowerShell에서 아래 명령으로 Up 상태를 저장한 뒤 다시 연결한다.

```powershell
docker ps --filter name=codyssey-repro-attach
docker attach codyssey-repro-attach
```

Bash에서 exit로 종료한 뒤 PowerShell의 `docker ps -a --filter name=codyssey-repro-attach`로 종료 상태를 비교한다.

## 5. 재현 자원 정리

다음 명령은 위에서 만든 재현용 컨테이너만 삭제한다.

```powershell
docker rm -f codyssey-repro-web codyssey-repro-bind codyssey-repro-after codyssey-repro-attach
```

볼륨 데이터까지 지워도 되는 경우에만 아래 명령을 실행한다. 삭제하면 재현용 hello.txt도 사라진다.

```powershell
docker volume rm codyssey-repro-data
```

이미 같은 이름의 컨테이너가 있으면 새 이름을 쓰거나 자신이 만든 자원을 확인하고 정리한다. 호스트 경로는 $PWD.Path를 사용하므로 복제한 위치에 맞춰진다. Linux Bash에서는 마운트 옵션을 `--mount "type=bind,source=$(pwd)/site,target=/usr/share/nginx/html,readonly"`로 바꿔 쓴다.


[README로 돌아가기](../README.md)
