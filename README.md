tugas-besar-if3130-sistem-paralel-dan-terdistribusi-{nama_kelompok}/
│
├── README.md                          # Cara build, run cluster, demo, daftar dependencies
├── DESIGN.md                          # Kontrak API, struktur log entry, aturan quorum, persistence
├── QNA_REFERENCES.md                  # Catatan klarifikasi library yang diizinkan
├── .gitignore
├── LICENSE
│
├── docs/
│   ├── api-contract.md                # RequestVote, AppendEntries, execute, request_log, add-node
│   ├── message-format.md              # Contoh payload JSON RPC antar node & client-server
│   ├── log-entry-format.md            # term, index, command/payload, metadata transaksi
│   ├── membership-rules.md            # Aturan quorum & kick node absen (n heartbeat)
│   ├── persistence-format.md          # Lokasi file, format data, recovery
│   ├── test-plan.md                   # Matriks T1–T14
│   └── demo-scenario.md               # Skenario demo lengkap
│
├── cmd/
│   ├── server/
│   │   └── main.go                    # Entry point node (konfigurasi node_id, addr, peer list)
│   └── client/
│       └── main.go                    # CLI client: ping/get/set/strln/del/append, request_log
│
├── internal/
│   │
│   ├── raft/                          # [RINO — Raft Core & Membership]
│   │   ├── node.go                    # State node: Follower/Candidate/Leader, term, votedFor
│   │   ├── election.go                # Election timeout, RequestVote, satu vote per term, Pre-Vote
│   │   ├── heartbeat.go               # Heartbeat periodik, deteksi timeout
│   │   ├── membership.go              # Add-node runtime, kick node absen n heartbeat
│   │   ├── quorum.go                  # Perhitungan quorum berbasis membership aktif
│   │   ├── state.go                   # Struktur state node + persist metadata Raft
│   │   ├── leader.go                  # Perilaku leader, step-down pada term baru
│   │   └── raft_test.go               # T1–T5, T13 (election, split vote, quorum, leader down)
│   │
│   ├── replication/                   # [FIRA — Log Replication & Storage]
│   │   ├── log.go                     # Log entries, index, term, commitIndex, lastApplied
│   │   ├── append_entries.go          # Kirim/terima AppendEntries, validasi prefix log
│   │   ├── commit.go                  # Aturan commit, apply ke state machine
│   │   ├── state_machine.go           # Key-value in-memory (get/set/strln/del/append/ping)
│   │   ├── transaction.go             # Batch atomic: validasi penuh sebelum mutasi (all-or-nothing)
│   │   ├── persistence.go             # Simpan log/state persisten (file/db)
│   │   ├── recovery.go                # Restart/rejoin, catch-up log/state
│   │   └── replication_test.go        # T6, T11, T12, T14 (replication, commit, batch, recovery)
│   │
│   ├── transport/                     # [ELI — RPC & Networking]
│   │   ├── rpc.go                     # Protokol RPC antar node (RequestVote, AppendEntries, add-node)
│   │   ├── client_rpc.go              # Handler execute & request_log dari client
│   │   ├── redirect.go                # Response leader_id/address saat request ke Follower
│   │   ├── codec.go                   # Serialization (JSON/protobuf) + contoh payload
│   │   └── transport_test.go          # Uji RPC & redirect
│   │
│   ├── client/                        # [ELI — Client Flow]
│   │   ├── cli.go                     # Parser command + string ber-spasi ("...")
│   │   ├── session.go                 # Simpan target koneksi ke Leader, redirect ulang transparan
│   │   ├── requester.go               # Kirim ulang request otomatis ke Leader
│   │   └── client_test.go             # T7–T9 (redirect, leader berubah, parser quote)
│   │
│   ├── logging/                       # [ELI — Logging]
│   │   ├── logger.go                  # Log ping & request (node, term, jenis, hasil, waktu)
│   │   └── logger_test.go             # T10
│   │
│   └── config/
│       └── config.go                  # Parsing konfigurasi node (id, addr, peer list, contact addr)
│
├── pkg/                               # (opsional) utilitas yang bisa dipakai bersama
│   ├── addr/
│   │   └── addr.go                    # Representasi Address (ip:port)
│   └── jsonutil/
│       └── json.go                    # Helper marshalling
│
├── deployments/                       # [ELI — Script/Docker]
│   ├── docker-compose.yml             # 4 node awal + client
│   ├── Dockerfile.server
│   ├── Dockerfile.client
│   ├── scripts/
│   │   ├── start-cluster.sh           # Jalankan 4 node (bridge/lo)
│   │   ├── stop-cluster.sh
│   │   ├── add-node.sh                # Tambah node runtime (pakai client add-node)
│   │   ├── remove-node.sh             # Kick node
│   │   └── tc-latency.sh              # Terapkan tc netem delay (untuk demo)
│   └── configs/
│       ├── node1.json
│       ├── node2.json
│       ├── node3.json
│       └── node4.json
│
├── tests/                             # [Tim — skenario end-to-end]
│   ├── e2e/
│   │   ├── t1_cluster_boot.sh
│   │   ├── t2_leader_failover.sh
│   │   ├── t3_quorum_node_down.sh
│   │   ├── t4_kick_absent_node.sh
│   │   ├── t5_add_node_runtime.sh
│   │   ├── t6_rejoin_sync.sh
│   │   ├── t7_redirect_from_follower.sh
│   │   ├── t8_leader_change_session.sh
│   │   ├── t9_quoted_string.sh
│   │   ├── t10_ping_request_log.sh
│   │   ├── t11_batch_valid.sh
│   │   ├── t12_batch_invalid_atomic.sh
│   │   ├── t13_partition_unstable.sh
│   │   └── t14_restart_cluster.sh
│   └── fixtures/
│       └── sample_commands.txt
│
├── dependencies/                      # Daftar dependency
│   ├── go.mod
│   ├── go.sum
│   └── DEPENDENCIES.md                # Jelaskan tiap lib: hanya transport/async/serialization/testing/logging
│
└── .gitlab-ci.yml                     # (opsional) CI: build + unit test