# Escalabilidade conceitual

Para atender 1 milhão de usuários e 100 mil conexões simultâneas no Brasil, seria necessário evoluir a implantação local para uma arquitetura distribuída. Réplicas stateless da API ficam atrás de um balanceador e escalam horizontalmente em Kubernetes ou Swarm. CDN entrega os arquivos estáticos perto dos usuários; Redis pode guardar dados de leitura curta, com invalidação após mudanças no cardápio. Filas desacoplam confirmação de pedidos e integrações.

O PostgreSQL pode usar pool de conexões (por exemplo, PgBouncer), réplicas de leitura e particionamento planejado conforme volume e região. Observabilidade reúne métricas, logs e traces; autoscaling ajusta réplicas por carga. Multi-região exige definir residência dos dados, consistência, failover e roteamento por região.

```mermaid
flowchart TB
  U[Usuários no Brasil] --> CDN[CDN: arquivos estáticos]
  U --> LB[Balanceador global]
  CDN --> LB
  LB --> API1[API stateless A]
  LB --> API2[API stateless B]
  LB --> APIN[API stateless N]
  API1 --> CACHE[Redis]
  API2 --> CACHE
  APIN --> CACHE
  API1 --> QUEUE[Fila de pedidos]
  API2 --> QUEUE
  APIN --> QUEUE
  QUEUE --> WORKERS[Workers escaláveis]
  API1 --> POOL[Pool de conexões]
  API2 --> POOL
  APIN --> POOL
  POOL --> DB[(PostgreSQL primário particionado)]
  DB --> REPLICA[(Réplicas de leitura)]
  API1 -. métricas, logs e traces .-> OBS[Observabilidade e autoscaling]
  WORKERS -. métricas .-> OBS
  DB -. métricas .-> OBS
```

Orquestração espalha as réplicas por zonas e regiões. Um desenho multi-região precisa de replicação e procedimentos de recuperação testados, além de monitoramento de latência e capacidade.
