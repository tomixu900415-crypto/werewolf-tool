<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>狼人杀身份抽取工具</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    
    <!-- 配置Tailwind自定义颜色和字体 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#6b21a8', // 主色调：紫色（代表神秘）
                        secondary: '#f59e0b', // 辅助色：琥珀色
                        dark: '#1e1b4b', // 深色背景
                        light: '#f5f3ff', // 浅色文本
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    
    <style type="text/tailwindcss">
        @layer utilities {
            .text-shadow {
                text-shadow: 0 2px 4px rgba(0,0,0,0.1);
            }
            .card-flip {
                transform-style: preserve-3d;
                transition: transform 0.6s;
            }
            .card-flip.flipped {
                transform: rotateY(180deg);
            }
            .card-front, .card-back {
                backface-visibility: hidden;
            }
            .card-back {
                transform: rotateY(180deg);
            }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-dark to-primary min-h-screen text-light font-sans">
    <!-- 页面容器 -->
    <div class="container mx-auto px-4 py-8 max-w-4xl">
        <!-- 标题区域 -->
        <header class="text-center mb-10">
            <h1 class="text-[clamp(2rem,5vw,3.5rem)] font-bold text-secondary mb-2 text-shadow">狼人杀身份抽取</h1>
            <p class="text-light/80 text-lg">公平、随机、防作弊的身份分配工具</p>
        </header>

        <!-- 主内容区 -->
        <main>
            <!-- 游戏创建/配置界面 (主持人) -->
            <section id="host-panel" class="bg-white/10 backdrop-blur-md rounded-xl p-6 mb-8 shadow-lg">
                <h2 class="text-2xl font-bold mb-6 flex items-center">
                    <i class="fa fa-cog text-secondary mr-2"></i> 游戏配置
                </h2>
                
                <div class="space-y-6">
                    <!-- 游戏人数设置 -->
                    <div>
                        <label for="player-count" class="block text-lg mb-2">参与人数</label>
                        <input type="number" id="player-count" min="4" max="20" value="12" 
                               class="w-full px-4 py-2 rounded-lg bg-white/5 border border-white/20 focus:border-secondary focus:outline-none focus:ring-2 focus:ring-secondary/50">
                    </div>
                    
                    <!-- 身份设置 -->
                    <div>
                        <h3 class="text-xl font-semibold mb-3">身份配置</h3>
                        <div id="roles-container" class="grid grid-cols-1 md:grid-cols-2 gap-3">
                            <!-- 身份选项会通过JS动态生成 -->
                        </div>
                    </div>
                    
                    <!-- 生成游戏按钮 -->
                    <button id="create-game" class="w-full bg-secondary hover:bg-secondary/90 text-dark font-bold py-3 px-6 rounded-lg transition-all duration-300 transform hover:scale-[1.02] active:scale-[0.98] flex items-center justify-center">
                        <i class="fa fa-magic mr-2"></i> 生成游戏链接
                    </button>
                </div>
                
                <!-- 游戏链接区域 (生成后显示) -->
                <div id="game-link-container" class="mt-6 hidden">
                    <div class="flex items-center">
                        <input type="text" id="game-link" readonly 
                               class="flex-1 px-4 py-2 rounded-l-lg bg-white/5 border border-r-0 border-white/20">
                        <button id="copy-link" class="bg-secondary text-dark px-4 py-2 rounded-r-lg hover:bg-secondary/90 transition-colors">
                            <i class="fa fa-copy"></i>
                        </button>
                    </div>
                    <p class="text-sm text-light/70 mt-2">
                        <i class="fa fa-info-circle mr-1"></i> 复制链接发送给玩家，他们输入自己的序号即可获取身份
                    </p>
                </div>
            </section>
            
            <!-- 玩家身份查看界面 -->
            <section id="player-panel" class="bg-white/10 backdrop-blur-md rounded-xl p-6 shadow-lg hidden">
                <h2 class="text-2xl font-bold mb-6 flex items-center">
                    <i class="fa fa-user-secret text-secondary mr-2"></i> 获取你的身份
                </h2>
                
                <div id="player-input-section" class="space-y-4">
                    <div>
                        <label for="player-number" class="block text-lg mb-2">请输入你的序号 (1-<span id="max-player">12</span>)</label>
                        <input type="number" id="player-number" min="1" max="12" value="1" 
                               class="w-full px-4 py-2 rounded-lg bg-white/5 border border-white/20 focus:border-secondary focus:outline-none focus:ring-2 focus:ring-secondary/50">
                    </div>
                    
                    <button id="get-identity" class="w-full bg-secondary hover:bg-secondary/90 text-dark font-bold py-3 px-6 rounded-lg transition-all duration-300 transform hover:scale-[1.02] active:scale-[0.98]">
                        <i class="fa fa-hand-paper-o mr-2"></i> 抽取身份
                    </button>
                </div>
                
                <!-- 身份卡片 (抽取后显示) -->
                <div id="identity-card" class="mt-8 relative h-64 hidden">
                    <div class="card-flip h-full">
                        <!-- 卡片正面 -->
                        <div class="card-front absolute inset-0 bg-gradient-to-br from-primary to-dark rounded-xl shadow-xl flex flex-col items-center justify-center border-4 border-secondary/50">
                            <i class="fa fa-question-circle text-6xl text-secondary mb-4"></i>
                            <p class="text-xl font-semibold">点击卡片查看身份</p>
                        </div>
                        
                        <!-- 卡片背面 -->
                        <div class="card-back absolute inset-0 bg-gradient-to-br from-dark to-primary rounded-xl shadow-xl flex flex-col items-center justify-center border-4 border-secondary/50 p-6 text-center">
                            <h3 id="role-name" class="text-3xl font-bold mb-2 text-secondary"></h3>
                            <p id="role-description" class="text-light/90"></p>
                        </div>
                    </div>
                </div>
                
                <!-- 错误提示 -->
                <div id="error-message" class="mt-4 p-4 bg-red-500/20 border border-red-500/40 rounded-lg text-red-300 hidden">
                    <i class="fa fa-exclamation-circle mr-2"></i>
                    <span id="error-text"></span>
                </div>
            </section>
        </main>
        
        <!-- 页脚 -->
        <footer class="mt-12 text-center text-light/60 text-sm">
            <p>狼人杀身份抽取工具 &copy; 2023</p>
        </footer>
    </div>

    <script>
        // 所有可用身份及其描述
        const allRoles = [
            { id: 'werewolf', name: '狼人', description: '每晚可以杀死一名玩家' },
            { id: 'seer', name: '预言家', description: '每晚可以查验一名玩家的身份' },
            { id: 'witch', name: '女巫', description: '拥有一瓶解药和一瓶毒药，每晚可以使用一次' },
            { id: 'hunter', name: '猎人', description: '被杀死时可以开枪带走一名玩家' },
            { id: 'white_wolf', name: '白神', description: '被投票出局时会翻牌，免疫此次放逐' },
            { id: 'guard', name: '守卫', description: '每晚可以守护一名玩家，防止其被狼人杀死' },
            { id: 'mechanical_wolf', name: '机械狼', description: '第一晚可以模仿一名玩家的身份并获得其能力' },
            { id: 'black_wolf_king', name: '黑狼王', description: '被投票出局时可以带走一名玩家' },
            { id: 'white_wolf_king', name: '白狼王', description: '可以在白天自爆并带走一名玩家' },
            { id: 'wolf_beauty', name: '狼美人', description: '每晚可以魅惑一名玩家，出局时被魅惑者一同死去' },
            { id: 'knight', name: '骑士', description: '白天可以决斗一名玩家，若对方是狼人则直接出局' },
            { id: 'dream_caster', name: '摄梦人', description: '每晚可以摄梦一名玩家，连续两晚摄梦同一人则其死亡' },
            { id: 'bear', name: '熊', description: '天亮时若左右两侧有狼人，则会咆哮' },
            { id: 'penguin', name: '企鹅', description: '每晚可以冰冻一名玩家，使其无法使用技能' },
            { id: 'villager', name: '村民', description: '没有特殊技能，只能参与投票' }
        ];

        // DOM元素
        const hostPanel = document.getElementById('host-panel');
        const playerPanel = document.getElementById('player-panel');
        const rolesContainer = document.getElementById('roles-container');
        const playerCountInput = document.getElementById('player-count');
        const createGameBtn = document.getElementById('create-game');
        const gameLinkContainer = document.getElementById('game-link-container');
        const gameLinkInput = document.getElementById('game-link');
        const copyLinkBtn = document.getElementById('copy-link');
        const playerNumberInput = document.getElementById('player-number');
        const maxPlayerSpan = document.getElementById('max-player');
        const getIdentityBtn = document.getElementById('get-identity');
        const identityCard = document.getElementById('identity-card');
        const cardFlip = document.querySelector('.card-flip');
        const roleName = document.getElementById('role-name');
        const roleDescription = document.getElementById('role-description');
        const errorMessage = document.getElementById('error-message');
        const errorText = document.getElementById('error-text');

        // 生成身份选项
        function generateRoleOptions() {
            rolesContainer.innerHTML = '';
            
            allRoles.forEach(role => {
                const roleDiv = document.createElement('div');
                roleDiv.className = 'flex items-center justify-between bg-white/5 p-3 rounded-lg';
                
                roleDiv.innerHTML = `
                    <div class="flex items-center">
                        <input type="checkbox" id="role-${role.id}" class="mr-3 h-5 w-5 accent-secondary" checked>
                        <label for="role-${role.id}" class="text-lg">${role.name}</label>
                    </div>
                    <div>
                        <label for="count-${role.id}" class="mr-2">数量:</label>
                        <input type="number" id="count-${role.id}" min="0" max="10" value="${role.id === 'villager' ? 4 : 1}" 
                               class="w-16 px-2 py-1 rounded bg-white/5 border border-white/20">
                    </div>
                `;
                
                rolesContainer.appendChild(roleDiv);
            });
        }

        // 生成唯一游戏ID
        function generateGameId() {
            const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
            let id = '';
            for (let i = 0; i < 8; i++) {
                id += chars.charAt(Math.floor(Math.random() * chars.length));
            }
            return id;
        }

        // 创建游戏
        function createGame() {
            const playerCount = parseInt(playerCountInput.value);
            const selectedRoles = [];
            
            // 收集选中的身份及其数量
            allRoles.forEach(role => {
                const checkbox = document.getElementById(`role-${role.id}`);
                const countInput = document.getElementById(`count-${role.id}`);
                
                if (checkbox.checked && parseInt(countInput.value) > 0) {
                    for (let i = 0; i < parseInt(countInput.value); i++) {
                        selectedRoles.push({
                            id: role.id,
                            name: role.name,
                            description: role.description
                        });
                    }
                }
            });
            
            // 验证身份总数是否与玩家数量匹配
            if (selectedRoles.length !== playerCount) {
                showError(`身份总数 (${selectedRoles.length}) 与玩家数量 (${playerCount}) 不匹配`);
                return;
            }
            
            // 打乱身份顺序
            const shuffledRoles = shuffleArray(selectedRoles);
            
            // 生成游戏ID
            const gameId = generateGameId();
            
            // 保存游戏数据到localStorage
            const gameData = {
                id: gameId,
                playerCount: playerCount,
                roles: shuffledRoles,
                revealed: Array(playerCount).fill(false) // 记录哪些序号已查看身份
            };
            
            localStorage.setItem(`werewolf_game_${gameId}`, JSON.stringify(gameData));
            
            // 生成并显示游戏链接
            const gameLink = `${window.location.origin}${window.location.pathname}?game=${gameId}`;
            gameLinkInput.value = gameLink;
            gameLinkContainer.classList.remove('hidden');
            
            // 复制链接到剪贴板
            copyLinkBtn.addEventListener('click', () => {
                gameLinkInput.select();
                document.execCommand('copy');
                
                // 显示复制成功反馈
                const originalText = copyLinkBtn.innerHTML;
                copyLinkBtn.innerHTML = '<i class="fa fa-check"></i>';
                setTimeout(() => {
                    copyLinkBtn.innerHTML = originalText;
                }, 2000);
            });
        }

        // 玩家获取身份
        function getIdentity() {
            const gameId = getGameIdFromUrl();
            if (!gameId) {
                showError('无效的游戏链接');
                return;
            }
            
            const gameData = JSON.parse(localStorage.getItem(`werewolf_game_${gameId}`));
            if (!gameData) {
                showError('游戏不存在或已过期');
                return;
            }
            
            const playerNumber = parseInt(playerNumberInput.value);
            
            // 验证玩家序号
            if (playerNumber < 1 || playerNumber > gameData.playerCount) {
                showError(`请输入1到${gameData.playerCount}之间的数字`);
                return;
            }
            
            // 检查是否已查看过
            if (gameData.revealed[playerNumber - 1]) {
                showError('该序号已查看过身份，不能重复查看');
                return;
            }
            
            // 记录已查看
            gameData.revealed[playerNumber - 1] = true;
            localStorage.setItem(`werewolf_game_${gameId}`, JSON.stringify(gameData));
            
            // 显示身份卡片
            const role = gameData.roles[playerNumber - 1];
            roleName.textContent = role.name;
            roleDescription.textContent = role.description;
            
            identityCard.classList.remove('hidden');
            hideError();
            
            // 点击卡片翻转
            cardFlip.classList.remove('flipped');
            setTimeout(() => {
                cardFlip.classList.add('flipped');
            }, 100);
        }

        // 从URL获取游戏ID
        function getGameIdFromUrl() {
            const params = new URLSearchParams(window.location.search);
            return params.get('game');
        }

        // 显示错误信息
        function showError(message) {
            errorText.textContent = message;
            errorMessage.classList.remove('hidden');
        }

        // 隐藏错误信息
        function hideError() {
            errorMessage.classList.add('hidden');
        }

        // 打乱数组顺序 (Fisher-Yates 洗牌算法)
        function shuffleArray(array) {
            const newArray = [...array];
            for (let i = newArray.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [newArray[i], newArray[j]] = [newArray[j], newArray[i]];
            }
            return newArray;
        }

        // 初始化页面
        function init() {
            // 生成身份选项
            generateRoleOptions();
            
            // 检查URL中是否有游戏ID
            const gameId = getGameIdFromUrl();
            
            if (gameId) {
                // 玩家模式
                hostPanel.classList.add('hidden');
                playerPanel.classList.remove('hidden');
                
                // 获取游戏数据
                const gameData = JSON.parse(localStorage.getItem(`werewolf_game_${gameId}`));
                if (gameData) {
                    // 设置最大玩家数
                    playerNumberInput.max = gameData.playerCount;
                    maxPlayerSpan.textContent = gameData.playerCount;
                } else {
                    showError('游戏不存在或已过期');
                }
            } else {
                // 主持人模式
                hostPanel.classList.remove('hidden');
                playerPanel.classList.add('hidden');
            }
            
            // 事件监听
            createGameBtn.addEventListener('click', createGame);
            getIdentityBtn.addEventListener('click', getIdentity);
            
            // 监听玩家数量变化，更新村民数量建议
            playerCountInput.addEventListener('input', () => {
                const playerCount = parseInt(playerCountInput.value) || 0;
                // 简单计算建议的村民数量
                const villagerCount = Math.max(1, playerCount - 7); // 假设其他身份总共7个左右
                document.getElementById('count-villager').value = villagerCount;
            });
        }

        // 页面加载完成后初始化
        document.addEventListener('DOMContentLoaded', init);
    </script>
</body>
</html>

