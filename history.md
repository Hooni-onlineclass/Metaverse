# 📄 WorkAdventure 최종 구축 기록 (history.md)

## 📅 마지막 작업: 2026-02-24
## 🎯 현황: 맵 로드 성공 / "방 연결 중" 단계에서 대기 중
## ⚙️ 최종 docker-compose.yaml
services:
  reverse-proxy:
    image: traefik:v3.6.1
    command:
      - --log.level=DEBUG
      - --providers.docker
      - --providers.docker.exposedbydefault=false
      - --entryPoints.web.address=:80
    ports:
      - "8081:80"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
    networks:
      - zep_network
  play:
    image: thecodingmachine/workadventure-play:master
    environment:
      - SECRET_KEY=your_secret_key
      - MAP_STORAGE_API_TOKEN=supersecrettoken
      - PUSHER_URL=https://zep.hooni.xyz/
      - FRONT_URL=/
      - INTERNAL_MAP_STORAGE_URL=http://map-storage:3000
      - PUBLIC_MAP_STORAGE_URL=https://zep.hooni.xyz
      - START_ROOM_URL=/_/global/zep.hooni.xyz/starter/map.json
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.play.rule=Host(`zep.hooni.xyz`)"
      - "traefik.http.routers.play.priority=1"
    restart: always
    networks:
      - zep_network
  back:
    image: thecodingmachine/workadventure-back:master
    environment:
      - PLAY_URL=https://zep.hooni.xyz
      - SECRET_KEY=your_secret_key
      - REDIS_HOST=redis
      - MAP_STORAGE_URL=map-storage:50053
    restart: always
    networks:
      - zep_network
  map-storage:
    image: thecodingmachine/workadventure-map-storage:master
    environment:
      - MAP_STORAGE_API_TOKEN=supersecrettoken
      - PUSHER_URL=https://zep.hooni.xyz/
      - AUTH_MAP_STORAGE_URL=http://map-storage:3000
      - API_URL=back:50051
    volumes:
      - map-storage-data:/maps
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.map-storage.rule=Host(`zep.hooni.xyz`) && PathPrefix(`/starter`)"
      - "traefik.http.routers.map-storage.priority=100"
      - "traefik.http.services.map-storage.loadbalancer.server.port=3000"
    restart: always
    networks:
      - zep_network
  redis:
    image: redis:6
    restart: always
    networks:
      - zep_network
volumes:
  map-storage-data:
networks:
  zep_network:
    external: true
sudo docker exec -u root zep-map-storage-1 mkdir -p /maps/starter

sudo docker exec -u root zep-map-storage-1 chmod -R 777 /maps

(map.json 주입 명령어 다시 실행)
EOF

📄 WorkAdventure_Setup_Status.md
📅 작업 일시: 2026-02-24
🛠 현재 상태
맵 로드 오류 해결: map.json 파일 형식을 올바르게 주입하여 toLowerCase 에러를 해결했습니다.

경로 단순화: map-storage라는 복잡한 경로를 걷어내고 zep.hooni.xyz/starter/map.json으로 바로 통신하도록 설정했습니다.

서버 응답 확인: 브라우저에서 지도를 요청했을 때 서버가 응답을 시작하여 "연결 중..." 단계까지 진입했습니다.



⏭️ 다음에 이어서 할 작업 (TODO)
Pusher 연결 확인: 현재 "연결 중..."에서 멈추는 것은 play 서버와 back 서버(또는 Pusher) 사이의 통신 지연일 가능성이 큽니다.

로그 모니터링: sudo docker logs zep-play-1 명령어로 "연결 중" 단계에서 발생하는 에러 메시지를 추적해야 합니다.

포트 포워딩 재점검: 외부 포트 8081과 내부 80 포트 사이의 Traefik 라우팅이 완전히 끝까지 도달하는지 확인이 필요합니다.
