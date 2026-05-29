<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Analisis Aljabar Boolean - K-Medoids</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --bg: #0a0e14;
            --bg-secondary: #0f1419;
            --card: #141a22;
            --card-hover: #1a222d;
            --border: #1e2733;
            --fg: #e6e6e6;
            --muted: #6b7280;
            --accent: #00d9a5;
            --accent-dim: #00b386;
            --danger: #ff6b6b;
            --warning: #fbbf24;
            --info: #38bdf8;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Space Grotesk', sans-serif;
            background: var(--bg);
            color: var(--fg);
            min-height: 100vh;
            overflow-x: hidden;
        }
        .font-mono { font-family: 'JetBrains Mono', monospace; }

        /* Background Effects */
        .bg-grid {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background-image: 
                linear-gradient(rgba(0, 217, 165, 0.03) 1px, transparent 1px),
                linear-gradient(90deg, rgba(0, 217, 165, 0.03) 1px, transparent 1px);
            background-size: 50px 50px;
            pointer-events: none; z-index: 0;
        }
        .bg-glow {
            position: fixed; width: 600px; height: 600px;
            border-radius: 50%; filter: blur(150px); opacity: 0.15;
            pointer-events: none; z-index: 0;
        }
        .glow-1 { top: -200px; right: -100px; background: var(--accent); }
        .glow-2 { bottom: -200px; left: -100px; background: #38bdf8; }

        /* Animations */
        @keyframes fadeInUp { from { opacity: 0; transform: translateY(30px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-10px); } }
        .animate-fade-up { animation: fadeInUp 0.6s ease-out forwards; }
        .animate-float { animation: float 3s ease-in-out infinite; }
        .stagger-1 { animation-delay: 0.1s; opacity: 0; }
        .stagger-2 { animation-delay: 0.2s; opacity: 0; }
        .stagger-3 { animation-delay: 0.3s; opacity: 0; }
        .stagger-4 { animation-delay: 0.4s; opacity: 0; }
        .stagger-5 { animation-delay: 0.5s; opacity: 0; }

        @media (prefers-reduced-motion: reduce) {
            *, *::before, *::after { animation-duration: 0.01ms !important; animation-iteration-count: 1 !important; transition-duration: 0.01ms !important; }
        }

        /* Components */
        .btn { padding: 12px 24px; border-radius: 8px; font-weight: 500; transition: all 0.2s ease; cursor: pointer; border: none; outline: none; }
        .btn:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
        .btn-primary { background: var(--accent); color: var(--bg); }
        .btn-primary:hover { background: var(--accent-dim); transform: translateY(-2px); }
        .btn-secondary { background: transparent; color: var(--fg); border: 1px solid var(--border); }
        .btn-secondary:hover { background: var(--card); border-color: var(--accent); }
        
        .card {
            background: var(--card); border: 1px solid var(--border);
            border-radius: 12px; padding: 24px; transition: all 0.3s ease;
        }
        .card:hover { border-color: rgba(0, 217, 165, 0.3); background: var(--card-hover); }

        .input-field {
            width: 100%; padding: 14px 16px; background: var(--bg-secondary);
            border: 1px solid var(--border); border-radius: 8px; color: var(--fg);
            font-family: inherit; transition: all 0.2s ease;
        }
        .input-field:focus { outline: none; border-color: var(--accent); box-shadow: 0 0 0 3px rgba(0, 217, 165, 0.1); }
        .input-field::placeholder { color: var(--muted); }

        /* Quiz Styles */
        .quiz-option {
            padding: 16px 20px; background: var(--bg-secondary);
            border: 2px solid var(--border); border-radius: 10px; cursor: pointer;
            transition: all 0.2s ease; text-align: left;
        }
        .quiz-option:hover { border-color: var(--accent); background: var(--card); }
        .quiz-option.correct { border-color: var(--accent); background: rgba(0, 217, 165, 0.2); }
        .quiz-option.wrong { border-color: var(--danger); background: rgba(255, 107, 107, 0.2); }

        .progress-bar { height: 6px; background: var(--border); border-radius: 3px; overflow: hidden; }
        .progress-fill { height: 100%; background: linear-gradient(90deg, var(--accent), #38bdf8); border-radius: 3px; transition: width 0.5s ease; }

        /* Code Block */
        .code-block { background: #0d1117; border: 1px solid var(--border); border-radius: 10px; overflow: hidden; }
        .code-header { background: var(--card); padding: 12px 16px; border-bottom: 1px solid var(--border); display: flex; align-items: center; gap: 8px; }
        .code-dot { width: 12px; height: 12px; border-radius: 50%; }
        .code-content { padding: 20px; overflow-x: auto; font-size: 13px; line-height: 1.6; }

        /* Stats Card */
        .stat-card { text-align: center; padding: 20px; }
        .stat-value {
            font-size: 2.5rem; font-weight: 700;
            background: linear-gradient(135deg, var(--accent), #38bdf8);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;
        }

        /* Navigation */
        .nav-item {
            padding: 10px 16px; border-radius: 8px; color: var(--muted);
            transition: all 0.2s ease; cursor: pointer;
        }
        .nav-item:hover { color: var(--fg); background: var(--card); }
        .nav-item.active { color: var(--accent); background: rgba(0, 217, 165, 0.1); }

        /* Page Transitions */
        .page { display: none; min-height: 100vh; }
        .page.active { display: block; }

        /* Scrollbar */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: var(--bg); }
        ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: var(--muted); }

        /* Cluster Badge */
        .cluster-badge { display: inline-flex; align-items: center; gap: 6px; padding: 4px 12px; border-radius: 20px; font-size: 12px; font-weight: 500; }
        .cluster-high { background: rgba(0, 217, 165, 0.2); color: var(--accent); }
        .cluster-medium { background: rgba(251, 191, 36, 0.2); color: var(--warning); }
        .cluster-low { background: rgba(255, 107, 107, 0.2); color: var(--danger); }

        /* Table Styles */
        .data-table { width: 100%; border-collapse: collapse; }
        .data-table th, .data-table td { padding: 14px 16px; text-align: left; border-bottom: 1px solid var(--border); }
        .data-table th { color: var(--muted); font-weight: 500; font-size: 12px; text-transform: uppercase; letter-spacing: 0.5px; }
        .data-table tr:hover td { background: var(--card); }

        /* Tab Toggle */
        .tab-btn { flex: 1; padding: 10px; border-radius: 6px; font-weight: 500; transition: all 0.3s ease; cursor: pointer; background: transparent; color: var(--muted); border: none; }
        .tab-btn.active { background: var(--accent); color: var(--bg); }

        /* Report Specific Styles */
        .report-title { border-bottom: 2px solid var(--accent); padding-bottom: 1rem; margin-bottom: 2rem; }
        .report-section { margin-bottom: 3rem; }
        .report-list li { position: relative; padding-left: 1.5rem; margin-bottom: 0.5rem; }
        .report-list li::before { content: "•"; color: var(--accent); position: absolute; left: 0; font-weight: bold; }
    </style>
</head>
<body>
    <!-- Background Effects -->
    <div class="bg-grid"></div>
    <div class="bg-glow glow-1"></div>
    <div class="bg-glow glow-2"></div>

    <!-- Login Page -->
    <div id="loginPage" class="page active">
        <div class="min-h-screen flex items-center justify-center p-6 relative z-10">
            <div class="w-full max-w-md">
                <!-- Logo -->
                <div class="text-center mb-10 animate-fade-up stagger-1">
                    <div class="inline-flex items-center justify-center w-20 h-20 rounded-2xl bg-gradient-to-br from-[#00d9a5] to-[#38bdf8] mb-6 animate-float">
                        <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="text-[#0a0e14]">
                            <path d="M12 2L2 7l10 5 10-5-10-5z"/>
                            <path d="M2 17l10 5 10-5"/>
                            <path d="M2 12l10 5 10-5"/>
                        </svg>
                    </div>
                    <h1 class="text-3xl font-bold mb-2">Analisis Aljabar Boolean</h1>
                    <p class="text-[var(--muted)]">Evaluasi Akademik dengan K-Medoids Clustering</p>
                </div>

                <!-- Auth Card -->
                <div class="card animate-fade-up stagger-2">
                    <!-- Tab Buttons -->
                    <div class="flex bg-[var(--bg-secondary)] rounded-lg p-1 mb-6">
                        <button id="loginTabBtn" class="tab-btn active" onclick="showAuthTab('login')">Login</button>
                        <button id="registerTabBtn" class="tab-btn" onclick="showAuthTab('register')">Daftar Tamu</button>
                    </div>

                    <!-- Login Form -->
                    <form id="loginForm" class="space-y-4">
                        <div>
                            <label class="block text-sm text-[var(--muted)] mb-2">Nama Pengguna</label>
                            <input type="text" id="loginUsername" class="input-field" placeholder="Masukkan nama pengguna" required>
                        </div>
                        
                        <div>
                            <label class="block text-sm text-[var(--muted)] mb-2">Kata Sandi</label>
                            <input type="password" id="loginPassword" class="input-field" placeholder="Masukkan kata sandi" required>
                        </div>

                        <div>
                            <label class="block text-sm text-[var(--muted)] mb-3">Masuk Sebagai</label>
                            <div class="grid grid-cols-2 gap-3">
                                <button type="button" class="role-btn p-4 rounded-lg border-2 border-[var(--border)] bg-[var(--bg-secondary)] transition-all hover:border-[var(--accent)] flex flex-col items-center gap-2" data-role="admin">
                                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="text-[var(--accent)]">
                                        <circle cx="12" cy="12" r="3"/>
                                        <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"/>
                                    </svg>
                                    <span class="font-medium">Admin</span>
                                </button>
                                <button type="button" class="role-btn p-4 rounded-lg border-2 border-[var(--border)] bg-[var(--bg-secondary)] transition-all hover:border-[var(--accent)] flex flex-col items-center gap-2" data-role="tamu">
                                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="text-[var(--info)]">
                                        <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/>
                                        <circle cx="12" cy="7" r="4"/>
                                    </svg>
                                    <span class="font-medium">Tamu</span>
                                </button>
                            </div>
                            <input type="hidden" id="selectedRole" value="">
                        </div>

                        <button type="submit" class="btn btn-primary w-full mt-6">Masuk</button>
                        <div id="loginError" class="hidden text-center text-sm text-[var(--danger)] mt-2"></div>
                    </form>

                    <!-- Register Form -->
                    <form id="registerForm" class="space-y-4 hidden">
                        <div>
                            <label class="block text-sm text-[var(--muted)] mb-2">Nama Pengguna Baru</label>
                            <input type="text" id="regUsername" class="input-field" placeholder="Buat nama pengguna" required>
                        </div>
                        <div>
                            <label class="block text-sm text-[var(--muted)] mb-2">Kata Sandi Baru</label>
                            <input type="password" id="regPassword" class="input-field" placeholder="Buat kata sandi" required>
                        </div>
                        <div>
                            <label class="block text-sm text-[var(--muted)] mb-2">Konfirmasi Kata Sandi</label>
                            <input type="password" id="regConfirmPassword" class="input-field" placeholder="Ulangi kata sandi" required>
                        </div>
                        <button type="submit" class="btn btn-primary w-full mt-6">Daftar sebagai Tamu</button>
                        <div id="registerMsg" class="hidden text-center text-sm mt-2"></div>
                    </form>

                    <div class="mt-6 pt-6 border-t border-[var(--border)]">
                        <p class="text-xs text-[var(--muted)] text-center">Admin: <span class="font-mono text-[var(--fg)]">admin</span> / <span class="font-mono text-[var(--fg)]">admin</span></p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Main Dashboard -->
    <div id="dashboardPage" class="page">
        <div class="flex min-h-screen relative z-10">
            <!-- Sidebar -->
            <aside class="w-64 bg-[var(--bg-secondary)] border-r border-[var(--border)] flex flex-col fixed h-full">
                <div class="p-6 border-b border-[var(--border)]">
                    <div class="flex items-center gap-3">
                        <div class="w-10 h-10 rounded-lg bg-gradient-to-br from-[#00d9a5] to-[#38bdf8] flex items-center justify-center">
                            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="text-[#0a0e14]">
                                <path d="M12 2L2 7l10 5 10-5-10-5z"/><path d="M2 17l10 5 10-5"/><path d="M2 12l10 5 10-5"/>
                            </svg>
                        </div>
                        <div>
                            <h2 class="font-semibold text-sm">Boolean Analysis</h2>
                            <p class="text-xs text-[var(--muted)]" id="userRoleBadge">Guest</p>
                        </div>
                    </div>
                </div>

                <nav class="flex-1 p-4 space-y-2">
                    <div class="nav-item active" data-page="overview" onclick="showSection('overview')">
                        <div class="flex items-center gap-3">
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="7" height="9"/><rect x="14" y="3" width="7" height="5"/><rect x="14" y="12" width="7" height="9"/><rect x="3" y="16" width="7" height="5"/></svg>
                            <span>Dashboard</span>
                        </div>
                    </div>
                    <div class="nav-item" data-page="quiz" onclick="showSection('quiz')">
                        <div class="flex items-center gap-3">
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
                            <span>Quiz Boolean</span>
                        </div>
                    </div>
                    <div class="nav-item" data-page="analysis" onclick="showSection('analysis')">
                        <div class="flex items-center gap-3">
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>
                            <span>Analisis Data</span>
                        </div>
                    </div>
                    <div class="nav-item" data-page="clustering" onclick="showSection('clustering')">
                        <div class="flex items-center gap-3">
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="3"/><circle cx="19" cy="5" r="2"/><circle cx="5" cy="5" r="2"/><circle cx="5" cy="19" r="2"/><circle cx="19" cy="19" r="2"/></svg>
                            <span>K-Medoids</span>
                        </div>
                    </div>
                    <div class="nav-item" data-page="report" onclick="showSection('report')">
                        <div class="flex items-center gap-3">
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>
                            <span>Laporan Riset</span>
                        </div>
                    </div>
                    <div class="nav-item" data-page="python" onclick="showSection('python')">
                        <div class="flex items-center gap-3">
                            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="16 18 22 12 16 6"/><polyline points="8 6 2 12 8 18"/></svg>
                            <span>Python Code</span>
                        </div>
                    </div>
                </nav>

                <div class="p-4 border-t border-[var(--border)]">
                    <button onclick="logout()" class="w-full btn btn-secondary text-sm flex items-center justify-center gap-2">
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" y1="12" x2="9" y2="12"/></svg>
                        Keluar
                    </button>
                </div>
            </aside>

            <!-- Main Content -->
            <main class="flex-1 ml-64 p-8">
                <!-- Overview Section (Dashboard) -->
                <section id="overviewSection" class="section-content">
                    <div class="mb-8 animate-fade-up">
                        <h1 class="text-3xl font-bold mb-2">Dashboard Overview</h1>
                        <p class="text-[var(--muted)]">Evaluasi Akademik Mahasiswa - Aljabar Boolean</p>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                        <div class="card stat-card animate-fade-up stagger-1">
                            <p class="text-[var(--muted)] text-sm mb-2">Total Responden</p>
                            <p class="stat-value">43</p>
                            <p class="text-xs text-[var(--muted)] mt-2">Mahasiswa PTIK-C</p>
                        </div>
                        <div class="card stat-card animate-fade-up stagger-2">
                            <p class="text-[var(--muted)] text-sm mb-2">Rata-rata Nilai</p>
                            <p class="stat-value">76.74</p>
                            <p class="text-xs text-[var(--muted)] mt-2">Kategori Cukup</p>
                        </div>
                        <div class="card stat-card animate-fade-up stagger-3">
                            <p class="text-[var(--muted)] text-sm mb-2">Median</p>
                            <p class="stat-value">80</p>
                            <p class="text-xs text-[var(--muted)] mt-2">Nilai Tengah</p>
                        </div>
                        <div class="card stat-card animate-fade-up stagger-4">
                            <p class="text-[var(--muted)] text-sm mb-2">Rentang</p>
                            <p class="stat-value">80</p>
                            <p class="text-xs text-[var(--muted)] mt-2">20 - 100</p>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
                        <div class="card animate-fade-up stagger-3">
                            <h3 class="font-semibold mb-4">Distribusi Nilai Mahasiswa</h3>
                            <canvas id="distributionChart" height="250"></canvas>
                        </div>
                        <div class="card animate-fade-up stagger-4">
                            <h3 class="font-semibold mb-4">Hasil Clustering K-Medoids</h3>
                            <canvas id="clusterChart" height="250"></canvas>
                        </div>
                    </div>
                </section>

                <!-- Quiz Section -->
                <section id="quizSection" class="section-content hidden">
                    <div class="mb-8 animate-fade-up">
                        <h1 class="text-3xl font-bold mb-2">Quiz Aljabar Boolean</h1>
                        <p class="text-[var(--muted)]">Uji pemahaman Anda tentang operasi logika dan hukum Boolean</p>
                    </div>
                    <div class="card mb-6 animate-fade-up stagger-1">
                        <div class="flex items-center justify-between mb-3">
                            <span class="text-sm text-[var(--muted)]">Progress Quiz</span>
                            <span class="text-sm font-mono" id="quizProgress">1/10</span>
                        </div>
                        <div class="progress-bar">
                            <div class="progress-fill" id="progressFill" style="width: 10%"></div>
                        </div>
                    </div>
                    <div class="card animate-fade-up stagger-2" id="quizContainer">
                        <div id="quizQuestion" class="mb-6"></div>
                        <div id="quizOptions" class="space-y-3 mb-6"></div>
                        <div id="quizFeedback" class="hidden mb-4 p-4 rounded-lg"></div>
                        <div class="flex gap-4">
                            <button id="nextQuestionBtn" class="btn btn-primary flex-1" onclick="nextQuestion()" disabled>Pertanyaan Selanjutnya</button>
                        </div>
                    </div>
                    <div id="quizResult" class="card hidden text-center">
                        <div class="py-8">
                            <div class="w-24 h-24 mx-auto mb-6 rounded-full bg-gradient-to-br from-[#00d9a5] to-[#38bdf8] flex items-center justify-center">
                                <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="text-[#0a0e14]"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/></svg>
                            </div>
                            <h2 class="text-2xl font-bold mb-2">Quiz Selesai</h2>
                            <p class="text-[var(--muted)] mb-6">Berikut adalah hasil quiz Anda</p>
                            <div class="inline-block p-6 bg-[var(--bg-secondary)] rounded-xl">
                                <p class="text-4xl font-bold text-[var(--accent)]" id="finalScore">0</p>
                                <p class="text-sm text-[var(--muted)]">dari 10 pertanyaan benar</p>
                            </div>
                        </div>
                        <button class="btn btn-primary" onclick="restartQuiz()">Ulangi Quiz</button>
                    </div>
                </section>

                <!-- Analysis Section -->
                <section id="analysisSection" class="section-content hidden">
                    <div class="mb-8 animate-fade-up">
                        <h1 class="text-3xl font-bold mb-2">Analisis Data</h1>
                        <p class="text-[var(--muted)]">Statistik deskriptif dan interpretasi hasil evaluasi</p>
                    </div>
                    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
                        <div class="card animate-fade-up stagger-1">
                            <h3 class="font-semibold mb-4">Statistik Deskriptif</h3>
                            <table class="data-table">
                                <thead><tr><th>Komponen</th><th>Nilai</th></tr></thead>
                                <tbody>
                                    <tr><td>Jumlah Responden</td><td class="font-mono text-[var(--accent)]">43</td></tr>
                                    <tr><td>Rata-rata (Mean)</td><td class="font-mono text-[var(--accent)]">76.74</td></tr>
                                    <tr><td>Median</td><td class="font-mono text-[var(--accent)]">80</td></tr>
                                    <tr><td>Nilai Tertinggi</td><td class="font-mono text-[var(--accent)]">100</td></tr>
                                    <tr><td>Nilai Terendah</td><td class="font-mono text-[var(--accent)]">20</td></tr>
                                </tbody>
                            </table>
                        </div>
                        <div class="card animate-fade-up stagger-2">
                            <h3 class="font-semibold mb-4">Pola Kesalahan Mahasiswa</h3>
                            <canvas id="errorChart" height="250"></canvas>
                        </div>
                    </div>
                </section>

                <!-- Clustering Section -->
                <section id="clusteringSection" class="section-content hidden">
                    <div class="mb-8 animate-fade-up">
                        <h1 class="text-3xl font-bold mb-2">K-Medoids Clustering</h1>
                        <p class="text-[var(--muted)]">Pengelompokan mahasiswa berdasarkan tingkat pemahaman</p>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
                        <div class="card animate-fade-up stagger-1 border-l-4 border-[var(--accent)]">
                            <div class="flex items-center justify-between mb-4">
                                <span class="cluster-badge cluster-high">Cluster 1</span>
                                <span class="text-2xl font-bold text-[var(--accent)]">18</span>
                            </div>
                            <h3 class="font-semibold mb-2">Tingkat Tinggi</h3>
                            <p class="text-sm text-[var(--muted)]">Rentang nilai: 80-100</p>
                        </div>
                        <div class="card animate-fade-up stagger-2 border-l-4 border-[var(--warning)]">
                            <div class="flex items-center justify-between mb-4">
                                <span class="cluster-badge cluster-medium">Cluster 2</span>
                                <span class="text-2xl font-bold text-[var(--warning)]">17</span>
                            </div>
                            <h3 class="font-semibold mb-2">Tingkat Sedang</h3>
                            <p class="text-sm text-[var(--muted)]">Rentang nilai: 60-79</p>
                        </div>
                        <div class="card animate-fade-up stagger-3 border-l-4 border-[var(--danger)]">
                            <div class="flex items-center justify-between mb-4">
                                <span class="cluster-badge cluster-low">Cluster 3</span>
                                <span class="text-2xl font-bold text-[var(--danger)]">8</span>
                            </div>
                            <h3 class="font-semibold mb-2">Tingkat Rendah</h3>
                            <p class="text-sm text-[var(--muted)]">Rentang nilai: &lt;60</p>
                        </div>
                    </div>
                    <div class="card animate-fade-up stagger-4 mb-6">
                        <h3 class="font-semibold mb-4">Visualisasi Cluster</h3>
                        <canvas id="scatterChart" height="300"></canvas>
                    </div>
                </section>

                <!-- REPORT SECTION (NEW) -->
                <section id="reportSection" class="section-content hidden">
                    <div class="mb-8 animate-fade-up">
                        <h1 class="text-3xl font-bold mb-2">Laporan Mini Riset</h1>
                        <p class="text-[var(--muted)]">Evaluasi Akademik Mahasiswa Berbasis Analisis Kesalahan dan Penerapan Algoritma K-Medoids</p>
                    </div>

                    <!-- A. Halaman Judul -->
                    <div class="card mb-6 animate-fade-up stagger-1 text-center">
                        <div class="py-8">
                            <h2 class="text-2xl font-bold mb-4">MINI RISET</h2>
                            <h3 class="text-xl text-[var(--accent)] mb-6">"EVALUASI AKADEMIK MAHASISWA BERBASIS ANALISIS KESALAHAN DAN PENERAPAN ALGORITMA K-MEDOIDS PADA MATERI ALJABAR BOOLEAN"</h3>
                            <div class="text-sm text-[var(--muted)] mb-6">
                                <p>Dosen Pengampu:</p>
                                <p>Dr. Amirhud Dalimunthe, S.T., M.Kom.</p>
                                <p>Novialdi Ashari, S.Stat., M.T.I.</p>
                            </div>
                            <div class="border-t border-[var(--border)] pt-4 text-sm">
                                <p class="font-semibold mb-2">DISUSUN OLEH: KELOMPOK 1</p>
                                <p>Zaky Akmal Rizky (5253151002)</p>
                                <p>Rhazqa Fhildiray Taher (5253151007)</p>
                                <p>Pniel Jhonspandi Nababan (5253151013)</p>
                                <p>Monica Oceven Aritonang (5253151020)</p>
                                <p>Kaila Damara Carissa (5253151027)</p>
                            </div>
                            <div class="mt-4 text-xs text-[var(--muted)]">
                                <p>Kelas: PTIK-C 2025 | Mata Kuliah: Matematika Terapan</p>
                                <p>UNIVERSITAS NEGERI MEDAN</p>
                            </div>
                        </div>
                    </div>

                    <!-- B. Bab 1 Deskripsi Sistem -->
                    <div class="card mb-6 animate-fade-up stagger-2 report-section">
                        <h3 class="text-xl font-bold text-[var(--accent)] mb-4 report-title">Bab 1. Deskripsi Sistem</h3>
                        <div class="space-y-4 text-sm">
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Nama Aplikasi</h4>
                                <p class="text-[var(--muted)]">Web Analisis Evaluasi Akademik Aljabar Boolean dengan Metode K-Medoids.</p>
                            </div>
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Tujuan Aplikasi</h4>
                                <ul class="report-list text-[var(--muted)]">
                                    <li>Mengidentifikasi bentuk kesalahan mahasiswa dalam memahami materi Aljabar Boolean.</li>
                                    <li>Menerapkan algoritma K-Medoids untuk mengelompokkan tingkat pemahaman mahasiswa.</li>
                                    <li>Memberikan rekomendasi pembelajaran yang lebih efektif berdasarkan hasil analisis.</li>
                                    <li>Menyediakan platform quiz interaktif untuk evaluasi mandiri.</li>
                                </ul>
                            </div>
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Cara Kerja Aplikasi</h4>
                                <p class="text-[var(--muted)]">Pengguna login (Admin/Tamu) -> Dashboard menampilkan statistik -> Pengguna mengikuti Quiz -> Sistem menganalisis hasil -> Visualisasi clustering K-Medoids ditampilkan.</p>
                            </div>
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Manfaat Aplikasi</h4>
                                <p class="text-[var(--muted)]">Membantu dosen dalam menentukan strategi pembelajaran yang tepat sesuai karakteristik kemampuan mahasiswa, serta memberikan gambaran pola kesalahan secara objektif.</p>
                            </div>
                        </div>
                    </div>

                    <!-- C. Bab 2 Konsep Matematika -->
                    <div class="card mb-6 animate-fade-up stagger-3 report-section">
                        <h3 class="text-xl font-bold text-[var(--accent)] mb-4 report-title">Bab 2. Konsep Matematika yang Digunakan</h3>
                        <div class="space-y-4 text-sm">
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Aljabar Boolean</h4>
                                <p class="text-[var(--muted)]">Operasi logika dasar (AND, OR, NOT), Hukum De Morgan, Penyederhanaan ekspresi logika. Materi ini menjadi fondasi dalam memahami proses pengambilan keputusan komputer.</p>
                            </div>
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Statistik Deskriptif</h4>
                                <p class="text-[var(--muted)]">Perhitungan Mean, Median, Modus, dan Rentang data untuk menganalisis distribusi nilai mahasiswa.</p>
                            </div>
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Algoritma K-Medoids</h4>
                                <div class="code-block mt-2 p-4 text-xs font-mono">
                                    <p>1. Tentukan jumlah cluster (k).</p>
                                    <p>2. Pilih medoid awal secara random.</p>
                                    <p>3. Hitung jarak antar data (Manhattan Distance).</p>
                                    <p>4. Kelompokkan data berdasarkan jarak terdekat ke medoid.</p>
                                    <p>5. Update medoid baru jika ditemukan jarak yang lebih minimum.</p>
                                    <p>6. Ulangi hingga tidak ada perubahan medoid (konvergen).</p>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- D. Bab 3 Implementasi Program -->
                    <div class="card mb-6 animate-fade-up stagger-4 report-section">
                        <h3 class="text-xl font-bold text-[var(--accent)] mb-4 report-title">Bab 3. Implementasi Program</h3>
                        <div class="space-y-4 text-sm">
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Tampilan Aplikasi</h4>
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mt-2">
                                    <div class="p-3 bg-[var(--bg-secondary)] rounded-lg">
                                        <p class="font-medium text-[var(--accent)]">1. Login Page</p>
                                        <p class="text-[var(--muted)]">Halaman autentikasi pengguna dengan pilihan Admin dan Tamu.</p>
                                    </div>
                                    <div class="p-3 bg-[var(--bg-secondary)] rounded-lg">
                                        <p class="font-medium text-[var(--accent)]">2. Dashboard</p>
                                        <p class="text-[var(--muted)]">Menampilkan ringkasan statistik dan grafik distribusi nilai.</p>
                                    </div>
                                    <div class="p-3 bg-[var(--bg-secondary)] rounded-lg">
                                        <p class="font-medium text-[var(--accent)]">3. Quiz Boolean</p>
                                        <p class="text-[var(--muted)]">Evaluasi interaktif 10 soal tentang Aljabar Boolean.</p>
                                    </div>
                                    <div class="p-3 bg-[var(--bg-secondary)] rounded-lg">
                                        <p class="font-medium text-[var(--accent)]">4. K-Medoids Visualizer</p>
                                        <p class="text-[var(--muted)]">Visualisasi hasil clustering menggunakan Chart.js.</p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- E. Bab 4 Hasil dan Kesimpulan -->
                    <div class="card mb-6 animate-fade-up stagger-5 report-section">
                        <h3 class="text-xl font-bold text-[var(--accent)] mb-4 report-title">Bab 4. Hasil dan Kesimpulan</h3>
                        <div class="space-y-4 text-sm">
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Hasil Pengujian</h4>
                                <div class="overflow-x-auto">
                                    <table class="data-table mt-2">
                                        <thead><tr><th>Parameter</th><th>Hasil</th></tr></thead>
                                        <tbody>
                                            <tr><td>Jumlah Responden</td><td>43 Mahasiswa</td></tr>
                                            <tr><td>Rata-rata Nilai</td><td>76.74</td></tr>
                                            <tr><td>Median</td><td>80</td></tr>
                                            <tr><td>Cluster 1 (Tinggi)</td><td>18 Mahasiswa (Rentang 80-100)</td></tr>
                                            <tr><td>Cluster 2 (Sedang)</td><td>17 Mahasiswa (Rentang 60-79)</td></tr>
                                            <tr><td>Cluster 3 (Rendah)</td><td>8 Mahasiswa (Rentang &lt;60)</td></tr>
                                        </tbody>
                                    </table>
                                </div>
                                <p class="text-[var(--muted)] mt-2">Kesalahan dominan: Penerapan Hukum De Morgan (35%) dan Penyederhanaan Ekspresi (28%).</p>
                            </div>
                            <div>
                                <h4 class="font-semibold text-[var(--fg)]">Kesimpulan</h4>
                                <p class="text-[var(--muted)]">Mahasiswa secara umum memiliki pemahaman cukup baik terhadap materi Aljabar Boolean. Algoritma K-Medoids efektif digunakan untuk pengelompokan tingkat pemahaman mahasiswa. Evaluasi akademik berbasis analisis data membantu meningkatkan efektivitas pembelajaran.</p>
                            </div>
                        </div>
                    </div>

                </section>

                <!-- Python Code Section -->
                <section id="pythonSection" class="section-content hidden">
                    <div class="mb-8 animate-fade-up">
                        <h1 class="text-3xl font-bold mb-2">Source Code Python</h1>
                        <p class="text-[var(--muted)]">Implementasi algoritma K-Medoids untuk analisis data</p>
                    </div>
                    <div class="code-block animate-fade-up stagger-1">
                        <div class="code-header">
                            <div class="code-dot bg-[#ff5f56]"></div>
                            <div class="code-dot bg-[#ffbd2e]"></div>
                            <div class="code-dot bg-[#27c93f]"></div>
                            <span class="ml-3 text-sm text-[var(--muted)]">kmedoids.py</span>
                        </div>
                        <pre class="code-content font-mono text-[var(--fg)]"><code>import numpy as np
from sklearn.metrics import pairwise_distances

class KMedoids:
    def __init__(self, n_clusters=3, max_iter=100, random_state=42):
        self.n_clusters = n_clusters
        self.max_iter = max_iter
        self.random_state = random_state
    
    def fit(self, X):
        np.random.seed(self.random_state)
        n_samples = X.shape[0]
        
        # Inisialisasi medoid
        medoid_indices = np.random.choice(n_samples, self.n_clusters, replace=False)
        self.medoids = X[medoid_indices]
        
        for iteration in range(self.max_iter):
            distances = pairwise_distances(X, self.medoids, metric='manhattan')
            new_labels = np.argmin(distances, axis=1)
            
            new_medoids = []
            for k in range(self.n_clusters):
                cluster_points = X[new_labels == k]
                if len(cluster_points) > 0:
                    cluster_distances = pairwise_distances(cluster_points, cluster_points, metric='manhattan')
                    medoid_idx = np.argmin(np.sum(cluster_distances, axis=1))
                    new_medoids.append(cluster_points[medoid_idx])
            
            if np.array_equal(self.medoids, new_medoids): break
            self.medoids = new_medoids
        
        self.labels = new_labels
        return self</code></pre>
                    </div>
                </section>
            </main>
        </div>
    </div>

    <script>
        // ==================== DATA ====================
        const quizData = [
            { question: "Hukum De Morgan menyatakan bahwa (A + B)' = ...", options: ["A' + B'", "A' . B'", "(A . B)'", "A + B'"], correct: 1, explanation: "Hukum De Morgan: (A + B)' = A' . B'" },
            { question: "Operasi AND dalam Aljabar Boolean dilambangkan dengan...", options: ["Simbol '+'", "Simbol '.' atau tanpa simbol", "Simbol '", "Simbol ⊕"], correct: 1, explanation: "Operasi AND dilambangkan dengan titik (.) atau tanpa simbol" },
            { question: "Berdasarkan hukum identitas, A + 0 = ...", options: ["0", "1", "A", "A'"], correct: 2, explanation: "Hukum identitas: A + 0 = A" },
            { question: "Hukum absorbsi menyatakan bahwa A + A.B = ...", options: ["A.B", "A", "B", "A + B"], correct: 1, explanation: "Hukum absorbsi: A + A.B = A" },
            { question: "Nilai dari A ⊕ A adalah...", options: ["A", "A'", "0", "1"], correct: 2, explanation: "XOR: A ⊕ A = 0" },
            { question: "Invers dari A dalam Aljabar Boolean adalah...", options: ["A", "A'", "0", "1"], correct: 1, explanation: "Invers (komplemen) dari A adalah A'" },
            { question: "Jika A = 1 dan B = 0, maka nilai A.B adalah...", options: ["0", "1", "Tidak terdefinisi", "A'"], correct: 0, explanation: "A.B = 1.0 = 0" },
            { question: "Hukum komutatif pada operasi AND menyatakan...", options: ["A.B = B.A", "A+B = B+A", "A.A = A", "A+1 = 1"], correct: 0, explanation: "Hukum komutatif: A.B = B.A" },
            { question: "Penyederhanaan A.A' menggunakan hukum...", options: ["Hukum identitas", "Hukum invers", "Hukum distributif", "Hukum absorbsi"], correct: 1, explanation: "A.A' = 0 (Hukum invers)" },
            { question: "Dalam K-Medoids, pusat cluster disebut...", options: ["Centroid", "Medoid", "Mean", "Mode"], correct: 1, explanation: "Pusat cluster pada K-Medoids disebut Medoid" }
        ];

        const studentData = [
            { id: 1, nilai: 80, cluster: 'high' }, { id: 2, nilai: 90, cluster: 'high' }, { id: 3, nilai: 100, cluster: 'high' },
            { id: 4, nilai: 80, cluster: 'high' }, { id: 5, nilai: 90, cluster: 'high' }, { id: 6, nilai: 70, cluster: 'medium' },
            { id: 7, nilai: 80, cluster: 'high' }, { id: 8, nilai: 90, cluster: 'high' }, { id: 9, nilai: 100, cluster: 'high' },
            { id: 10, nilai: 80, cluster: 'high' }, { id: 11, nilai: 70, cluster: 'medium' }, { id: 12, nilai: 60, cluster: 'medium' },
            { id: 13, nilai: 80, cluster: 'high' }, { id: 14, nilai: 90, cluster: 'high' }, { id: 15, nilai: 100, cluster: 'high' },
            { id: 16, nilai: 80, cluster: 'high' }, { id: 17, nilai: 70, cluster: 'medium' }, { id: 18, nilai: 60, cluster: 'medium' },
            { id: 19, nilai: 50, cluster: 'low' }, { id: 20, nilai: 80, cluster: 'high' }, { id: 21, nilai: 90, cluster: 'high' },
            { id: 22, nilai: 100, cluster: 'high' }, { id: 23, nilai: 80, cluster: 'high' }, { id: 24, nilai: 70, cluster: 'medium' },
            { id: 25, nilai: 60, cluster: 'medium' }, { id: 26, nilai: 80, cluster: 'high' }, { id: 27, nilai: 90, cluster: 'high' },
            { id: 28, nilai: 80, cluster: 'high' }, { id: 29, nilai: 70, cluster: 'medium' }, { id: 30, nilai: 100, cluster: 'high' },
            { id: 31, nilai: 80, cluster: 'high' }, { id: 32, nilai: 90, cluster: 'high' }, { id: 33, nilai: 80, cluster: 'high' },
            { id: 34, nilai: 20, cluster: 'low' }, { id: 35, nilai: 80, cluster: 'high' }, { id: 36, nilai: 90, cluster: 'high' },
            { id: 37, nilai: 100, cluster: 'high' }, { id: 38, nilai: 80, cluster: 'high' }, { id: 39, nilai: 70, cluster: 'medium' },
            { id: 40, nilai: 60, cluster: 'medium' }, { id: 41, nilai: 80, cluster: 'high' }, { id: 42, nilai: 90, cluster: 'high' },
            { id: 43, nilai: 40, cluster: 'low' }
        ];

        // ==================== STATE ====================
        let currentUser = null;
        let currentRole = null;
        let currentQuestion = 0;
        let score = 0;
        let selectedAnswer = null;

        // ==================== INITIALIZATION ====================
        document.addEventListener('DOMContentLoaded', function() {
            initRoleButtons();
            initAuthForms();
            if (!localStorage.getItem('registeredUsers')) {
                localStorage.setItem('registeredUsers', JSON.stringify([]));
            }
        });

        function showAuthTab(tab) {
            const loginForm = document.getElementById('loginForm');
            const registerForm = document.getElementById('registerForm');
            const loginTab = document.getElementById('loginTabBtn');
            const registerTab = document.getElementById('registerTabBtn');

            if (tab === 'login') {
                loginForm.classList.remove('hidden');
                registerForm.classList.add('hidden');
                loginTab.classList.add('active');
                registerTab.classList.remove('active');
            } else {
                loginForm.classList.add('hidden');
                registerForm.classList.remove('hidden');
                loginTab.classList.remove('active');
                registerTab.classList.add('active');
            }
        }

        function initRoleButtons() {
            const roleButtons = document.querySelectorAll('.role-btn');
            roleButtons.forEach(btn => {
                btn.addEventListener('click', function() {
                    roleButtons.forEach(b => b.classList.remove('border-[var(--accent)]', 'bg-[rgba(0,217,165,0.1)]'));
                    this.classList.add('border-[var(--accent)]', 'bg-[rgba(0,217,165,0.1)]');
                    document.getElementById('selectedRole').value = this.dataset.role;
                });
            });
        }

        function initAuthForms() {
            document.getElementById('loginForm').addEventListener('submit', function(e) {
                e.preventDefault();
                const username = document.getElementById('loginUsername').value;
                const password = document.getElementById('loginPassword').value;
                const role = document.getElementById('selectedRole').value;
                const errorEl = document.getElementById('loginError');

                if (!role) { errorEl.textContent = 'Pilih peran (Admin atau Tamu) terlebih dahulu!'; errorEl.classList.remove('hidden'); return; }

                if (role === 'admin') {
                    if (username === 'admin' && password === 'admin') { currentUser = username; currentRole = 'admin'; showDashboard(); }
                    else { errorEl.textContent = 'Username atau password Admin salah!'; errorEl.classList.remove('hidden'); }
                    return;
                }

                if (role === 'tamu') {
                    const users = JSON.parse(localStorage.getItem('registeredUsers'));
                    const foundUser = users.find(u => u.username === username && u.password === password);
                    if (foundUser) { currentUser = username; currentRole = 'tamu'; showDashboard(); }
                    else { errorEl.textContent = 'Username atau password Tamu salah, atau belum terdaftar.'; errorEl.classList.remove('hidden'); }
                }
            });

            document.getElementById('registerForm').addEventListener('submit', function(e) {
                e.preventDefault();
                const username = document.getElementById('regUsername').value;
                const password = document.getElementById('regPassword').value;
                const confirmPassword = document.getElementById('regConfirmPassword').value;
                const msgEl = document.getElementById('registerMsg');

                if (password !== confirmPassword) { msgEl.textContent = 'Konfirmasi kata sandi tidak cocok!'; msgEl.className = 'text-center text-sm mt-2 text-[var(--danger)]'; msgEl.classList.remove('hidden'); return; }

                let users = JSON.parse(localStorage.getItem('registeredUsers'));
                const exists = users.find(u => u.username === username);
                if (exists) { msgEl.textContent = 'Username sudah digunakan!'; msgEl.className = 'text-center text-sm mt-2 text-[var(--danger)]'; msgEl.classList.remove('hidden'); return; }

                users.push({ username, password });
                localStorage.setItem('registeredUsers', JSON.stringify(users));
                msgEl.textContent = 'Pendaftaran berhasil! Silakan login.'; msgEl.className = 'text-center text-sm mt-2 text-[var(--accent)]'; msgEl.classList.remove('hidden');
                setTimeout(() => { document.getElementById('regUsername').value = ''; document.getElementById('regPassword').value = ''; document.getElementById('regConfirmPassword').value = ''; showAuthTab('login'); msgEl.classList.add('hidden'); }, 1500);
            });
        }

        // ==================== NAVIGATION ====================
        function showDashboard() {
            document.getElementById('loginPage').classList.remove('active');
            document.getElementById('dashboardPage').classList.add('active');
            document.getElementById('userRoleBadge').textContent = currentRole === 'admin' ? 'Administrator' : 'Tamu';
            setTimeout(() => { initCharts(); }, 100);
            loadQuestion();
        }

        function logout() {
            currentUser = null; currentRole = null;
            document.getElementById('dashboardPage').classList.remove('active');
            document.getElementById('loginPage').classList.add('active');
            document.getElementById('loginForm').reset();
            document.getElementById('selectedRole').value = '';
            document.querySelectorAll('.role-btn').forEach(b => b.classList.remove('border-[var(--accent)]', 'bg-[rgba(0,217,165,0.1)]'));
            document.getElementById('loginError').classList.add('hidden');
            currentQuestion = 0; score = 0;
        }

        function showSection(sectionId) {
            document.querySelectorAll('.nav-item').forEach(item => item.classList.remove('active'));
            document.querySelector(`.nav-item[data-page="${sectionId}"]`).classList.add('active');
            document.querySelectorAll('.section-content').forEach(section => section.classList.add('hidden'));
            document.getElementById(`${sectionId}Section`).classList.remove('hidden');
        }

        // ==================== CHARTS ====================
        function initCharts() {
            initDistributionChart();
            initClusterChart();
            initErrorChart();
            initScatterChart();
        }

        function initDistributionChart() {
            const ctx = document.getElementById('distributionChart'); if (!ctx) return; if (ctx.chart) ctx.chart.destroy();
            const distribution = {};
            studentData.forEach(s => { const range = Math.floor(s.nilai / 10) * 10; distribution[range] = (distribution[range] || 0) + 1; });
            const labels = Object.keys(distribution).sort((a, b) => a - b).map(k => `${k}-${parseInt(k)+9}`);
            const data = Object.keys(distribution).sort((a, b) => a - b).map(k => distribution[k]);
            ctx.chart = new Chart(ctx, { type: 'bar', data: { labels: labels, datasets: [{ label: 'Jumlah Mahasiswa', data: data, backgroundColor: 'rgba(0, 217, 165, 0.6)', borderColor: 'rgba(0, 217, 165, 1)', borderWidth: 1, borderRadius: 6 }] }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { beginAtZero: true, grid: { color: 'rgba(255, 255, 255, 0.05)' }, ticks: { color: '#6b7280' } }, x: { grid: { display: false }, ticks: { color: '#6b7280' } } } } });
        }

        function initClusterChart() {
            const ctx = document.getElementById('clusterChart'); if (!ctx) return; if (ctx.chart) ctx.chart.destroy();
            const clusterCounts = { high: studentData.filter(s => s.cluster === 'high').length, medium: studentData.filter(s => s.cluster === 'medium').length, low: studentData.filter(s => s.cluster === 'low').length };
            ctx.chart = new Chart(ctx, { type: 'doughnut', data: { labels: ['Tinggi (80-100)', 'Sedang (60-79)', 'Rendah (<60)'], datasets: [{ data: [clusterCounts.high, clusterCounts.medium, clusterCounts.low], backgroundColor: ['rgba(0, 217, 165, 0.8)', 'rgba(251, 191, 36, 0.8)', 'rgba(255, 107, 107, 0.8)'], borderWidth: 0 }] }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom', labels: { color: '#6b7280', padding: 20 } } } } });
        }

        function initErrorChart() {
            const ctx = document.getElementById('errorChart'); if (!ctx) return; if (ctx.chart) ctx.chart.destroy();
            ctx.chart = new Chart(ctx, { type: 'bar', data: { labels: ['De Morgan', 'Penyederhanaan', 'Operasi Logika', 'Simbol/Notasi'], datasets: [{ label: 'Persentase Kesalahan', data: [35, 28, 22, 15], backgroundColor: ['rgba(255, 107, 107, 0.7)', 'rgba(251, 191, 36, 0.7)', 'rgba(56, 189, 248, 0.7)', 'rgba(0, 217, 165, 0.7)'], borderRadius: 6 }] }, options: { indexAxis: 'y', responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { x: { beginAtZero: true, max: 40, grid: { color: 'rgba(255, 255, 255, 0.05)' }, ticks: { color: '#6b7280', callback: v => v + '%' } }, y: { grid: { display: false }, ticks: { color: '#e6e6e6' } } } } });
        }

        function initScatterChart() {
            const ctx = document.getElementById('scatterChart'); if (!ctx) return; if (ctx.chart) ctx.chart.destroy();
            const clusterColors = { high: 'rgba(0, 217, 165, 0.7)', medium: 'rgba(251, 191, 36, 0.7)', low: 'rgba(255, 107, 107, 0.7)' };
            const datasets = ['high', 'medium', 'low'].map(cluster => {
                const data = studentData.filter(s => s.cluster === cluster).map(s => ({ x: s.id, y: s.nilai }));
                return { label: cluster === 'high' ? 'Tinggi' : cluster === 'medium' ? 'Sedang' : 'Rendah', data: data, backgroundColor: clusterColors[cluster], pointRadius: 8, pointHoverRadius: 12 };
            });
            ctx.chart = new Chart(ctx, { type: 'scatter', data: { datasets }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'top', labels: { color: '#6b7280' } } }, scales: { x: { title: { display: true, text: 'ID Mahasiswa', color: '#6b7280' }, grid: { color: 'rgba(255, 255, 255, 0.05)' }, ticks: { color: '#6b7280' } }, y: { title: { display: true, text: 'Nilai', color: '#6b7280' }, min: 0, max: 110, grid: { color: 'rgba(255, 255, 255, 0.05)' }, ticks: { color: '#6b7280' } } } } });
        }

        // ==================== QUIZ ====================
        function loadQuestion() {
            if (currentQuestion >= quizData.length) { showQuizResult(); return; }
            const q = quizData[currentQuestion];
            document.getElementById('quizProgress').textContent = `${currentQuestion + 1}/${quizData.length}`;
            document.getElementById('progressFill').style.width = `${((currentQuestion + 1) / quizData.length) * 100}%`;
            document.getElementById('quizQuestion').innerHTML = `<div class="flex items-start gap-4"><div class="w-10 h-10 rounded-lg bg-[var(--accent)] flex items-center justify-center text-[var(--bg)] font-bold flex-shrink-0">${currentQuestion + 1}</div><h3 class="text-xl font-medium">${q.question}</h3></div>`;
            document.getElementById('quizOptions').innerHTML = q.options.map((opt, idx) => `<button class="quiz-option w-full" onclick="selectAnswer(${idx})" data-index="${idx}"><div class="flex items-center gap-3"><span class="w-8 h-8 rounded-full border-2 border-[var(--border)] flex items-center justify-center text-sm font-mono">${String.fromCharCode(65 + idx)}</span><span>${opt}</span></div></button>`).join('');
            selectedAnswer = null;
            document.getElementById('quizFeedback').classList.add('hidden');
            document.getElementById('nextQuestionBtn').disabled = true;
            document.getElementById('nextQuestionBtn').textContent = 'Pilih Jawaban';
        }

        function selectAnswer(index) {
            if (selectedAnswer !== null) return;
            selectedAnswer = index;
            const q = quizData[currentQuestion];
            const options = document.querySelectorAll('.quiz-option');
            const feedbackEl = document.getElementById('quizFeedback');
            const nextBtn = document.getElementById('nextQuestionBtn');
            options.forEach((opt, idx) => { opt.classList.remove('selected'); if (idx === index) opt.classList.add('selected'); });
            feedbackEl.classList.remove('hidden');
            if (index === q.correct) {
                score++;
                options[index].classList.add('correct');
                feedbackEl.innerHTML = `<div class="flex items-start gap-3"><div><p class="font-medium text-[var(--accent)]">Benar!</p><p class="text-sm text-[var(--muted)] mt-1">${q.explanation}</p></div></div>`;
                feedbackEl.className = 'mb-4 p-4 rounded-lg bg-[rgba(0,217,165,0.1)] border border-[var(--accent)]';
            } else {
                options[index].classList.add('wrong');
                options[q.correct].classList.add('correct');
                feedbackEl.innerHTML = `<div class="flex items-start gap-3"><div><p class="font-medium text-[var(--danger)]">Salah!</p><p class="text-sm text-[var(--muted)] mt-1">Jawaban yang benar: ${String.fromCharCode(65 + q.correct)}. ${q.explanation}</p></div></div>`;
                feedbackEl.className = 'mb-4 p-4 rounded-lg bg-[rgba(255,107,107,0.1)] border border-[var(--danger)]';
            }
            nextBtn.disabled = false;
            nextBtn.textContent = currentQuestion === quizData.length - 1 ? 'Lihat Hasil' : 'Pertanyaan Selanjutnya';
        }

        function nextQuestion() {
            currentQuestion++;
            if (currentQuestion >= quizData.length) showQuizResult();
            else loadQuestion();
        }

        function showQuizResult() {
            document.getElementById('quizContainer').classList.add('hidden');
            document.getElementById('quizResult').classList.remove('hidden');
            document.getElementById('finalScore').textContent = `${score}/${quizData.length}`;
        }

        function restartQuiz() {
            currentQuestion = 0; score = 0; selectedAnswer = null;
            document.getElementById('quizContainer').classList.remove('hidden');
            document.getElementById('quizResult').classList.add('hidden');
            loadQuestion();
        }
    </script>
</body>
</html>
