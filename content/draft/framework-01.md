+++
title = "打造框架"
description = ""
date = 2025-05-27 22:00:00
author = "Rhan0"
tags=["framework"]
weight= 2
+++

## Framework

### 概要
- 打造 framework 的動機？ 
- 想解決什麼問題？
- 做法評估

### 打造 framework 的動機？
1. 升版導致內容壞掉
2. 製作內容不友善

### 想解決什麼問題？
TODO: 

## 做法評估

### 問題

1. framework 的做法如果是整包 Worlds codebase 都打包進去則會增加整體內容打包的大小，是否考慮提供多版本並存的 Worlds runtime 環境，為每個發布的內容標記所使用的 Worlds 版本，runtime codebase 由 Worlds 負責 host 就好？不用每個 content 都把 worlds 包進去 (降低存放空間壓力、成本?)
- 模組部分保留在平台，而不是每一個模組都做成靜態
- publish 的模組對象指向 cdn


2. 需要跟其他前端框架整合嗎？ 例如 react？

3. 預期 framework 使用方式為何？

### 挑戰 & 限制

1. 架構是綁定的

```javascript
// PlayCanvas 強制的 ECS 模式
var entity = new pc.Entity();
entity.addComponent('render');
entity.addComponent('script');

// 你無法改變這個基本結構
// 所有自定義功能都必須透過 Component 或 Script 實現
```

限制：
- 無法改變核心的 Entity-Component 關係
- 必須遵循 PlayCanvas 的生命週期管理
- 受限於既有的 Component 類型和接口

2. 框架層級衝突

如果你想建立的新框架有不同的設計理念：
```javascript
// 假設你想要實現 Actor-Component 模式（類似 Unreal）
class Actor {
    constructor() {
        this.components = new Map();
    }
}

// 但在 PlayCanvas 中，你仍然被限制在：
// Entity -> Components 的固定模式
```

3. 擴展性限制
- PlayCanvas 的核心系統（渲染、物理、音頻）無法大幅修改
- 插件系統有限，主要透過 Script Components
- 無法改變引擎的基礎行為

### 可行的發展方向

1. 領域特定框架 Domain-specific language(DSL)

在 PlayCanvas 上層建立特定用途的框架：

- 例如：UI 框架
```javascript
class UIFramework {
    static createPanel(entity, config) {
        entity.addComponent('element', {
            type: 'image',
            ...config
        });
        return new UIPanel(entity);
    }
}
```

- 例如：遊戲機制框架
```javascript
class GameplayFramework {
    static createCharacter(entity, stats) {
        entity.addComponent('script', {
            scripts: [CharacterController, HealthSystem]
        });
        return new Character(entity, stats);
    }
}
```

2. 工作流程框架

專注於開發流程和工具鏈：

- 例如：狀態管理框架
```javascript
class StateManager {
    constructor(app) {
        this.app = app;
        this.states = new Map();
    }
    
    transition(fromState, toState) {
        // 管理場景切換、資源載入等
    }
}
```

- 例如：資料驅動框架
```javascript
class DataDrivenFramework {
    static createFromConfig(config) {
        // 從 JSON 配置生成 PlayCanvas entities
    }
}
```

3. 抽象層框架

提供更高階的 API：
- 例如：簡化的遊戲物件系統

```javascript
class GameObject {
    constructor(name) {
        this.entity = new pc.Entity(name);
        this.behaviors = [];
    }
    
    addBehavior(behavior) {
        // 將 behavior 轉換為 PlayCanvas script
        this.entity.script.create(behavior.name, behavior.attributes);
    }
}
```

### 發展策略

1. 垂直整合 - 針對特定類型的遊戲或應用
```javascript
class PlatformerFramework {
    static setup(app) {
        // 預設的物理設定、相機系統、輸入處理
    }
    
    static createPlayer(config) {
        // 標準化的角色創建
    }
}
```

2. 橫向擴展 - 提供 PlayCanvas 缺少的功能
```javascript
// 進階動畫系統
class AnimationFramework {
    static timeline() {
        return new Timeline(); // 類似 GSAP
    }
}

// 狀態機系統
class StateMachine {
    constructor(entity) {
        this.entity = entity;
        this.states = new Map();
    }
}
```

3. 開發工具框架 - 提升開發體驗
```javascript
// 除錯和性能監控
class DevTools {
    static enableProfiler() {
        // 在 PlayCanvas 基礎上添加性能分析
    }
}

// 熱重載系統
class HotReload {
    static watch(scriptPath) {
        // 監控腳本變化並自動重載
    }
}
```

### 定義：內容創作導向的 3D 框架

1. 分層架構
```
┌─────────────────────────────────────┐
│        Content Creator API          │  <- 創作者使用的高階 API
├─────────────────────────────────────┤
│        Feature Modules              │  <- 功能模組層
│  Avatar | Network | Camera | UI     │
├─────────────────────────────────────┤
│        Core Framework               │  <- 核心框架層
│   Scene | Entity | Component        │
├─────────────────────────────────────┤
│        PlayCanvas Engine            │  <- 引擎層
└─────────────────────────────────────┘
```

2. 核心框架層 (Core Framework)

```javascript
// ===== 核心框架層 =====

class ContentFramework {
    constructor(canvasElement, options = {}) {
        this.app = new pc.Application(canvasElement, options);
        this.modules = new Map();
        this.scenes = new Map();
        this.currentScene = null;
        
        this.initializeCore();
    }
    
    initializeCore() {
        // 註冊核心模組
        this.registerModule('avatar', new AvatarModule(this));
        this.registerModule('network', new NetworkModule(this));
        this.registerModule('camera', new CameraModule(this));
        this.registerModule('ui', new UIModule(this));
        this.registerModule('decoration', new DecorationModule(this));
        this.registerModule('loading', new LoadingModule(this));
        
        // 設置事件系統
        this.eventBus = new EventBus();
    }
    
    registerModule(name, module) {
        this.modules.set(name, module);
        module.initialize();
    }
    
    getModule(name) {
        return this.modules.get(name);
    }
}

// ===== 場景管理系統 =====

class Scene3D {
    constructor(name, framework) {
        this.name = name;
        this.framework = framework;
        this.entities = new Map();
        this.config = {};
        this.initialized = false;
    }
    
    async load(config) {
        this.config = config;
        
        // 載入場景資源
        await this.loadAssets(config.assets || []);
        
        // 設置環境
        if (config.environment) {
            await this.setupEnvironment(config.environment);
        }
        
        // 創建預設物件
        if (config.defaultObjects) {
            this.createDefaultObjects(config.defaultObjects);
        }
        
        this.initialized = true;
        this.framework.eventBus.emit('scene:loaded', this);
    }
    
    async loadAssets(assets) {
        const promises = assets.map(asset => {
            return new Promise((resolve) => {
                this.framework.app.assets.load(asset, resolve);
            });
        });
        
        await Promise.all(promises);
    }
    
    setupEnvironment(envConfig) {
        // 設置天空盒、光照等
        if (envConfig.skybox) {
            // 設置天空盒
        }
        
        if (envConfig.lighting) {
            // 設置光照
        }
    }
    
    createEntity(name, components = []) {
        const entity = new pc.Entity(name);
        
        // 添加組件
        components.forEach(comp => {
            entity.addComponent(comp.type, comp.options);
        });
        
        this.entities.set(name, entity);
        this.framework.app.root.addChild(entity);
        
        return entity;
    }
}

// ===== 模組基類 =====

class BaseModule {
    constructor(framework) {
        this.framework = framework;
        this.app = framework.app;
        this.eventBus = framework.eventBus;
        this.initialized = false;
    }
    
    async initialize() {
        if (this.initialized) return;
        
        await this.setup();
        this.bindEvents();
        this.initialized = true;
    }
    
    async setup() {
        // 子類實現
    }
    
    bindEvents() {
        // 子類實現
    }
    
    destroy() {
        // 清理資源
    }
}

// ===== Avatar 模組 =====

class AvatarModule extends BaseModule {
    constructor(framework) {
        super(framework);
        this.avatars = new Map();
        this.defaultAvatar = null;
    }
    
    async setup() {
        // 預載入 VRM 相關資源
        await this.loadVRMSupport();
    }
    
    async loadVRMSupport() {
        // 載入 VRM 載入器
        // 設置預設動畫
    }
    
    async createAvatar(id, config) {
        const avatar = new Avatar(id, config);
        await avatar.load();
        
        this.avatars.set(id, avatar);
        this.eventBus.emit('avatar:created', avatar);
        
        return avatar;
    }
    
    getAvatar(id) {
        return this.avatars.get(id);
    }
    
    removeAvatar(id) {
        const avatar = this.avatars.get(id);
        if (avatar) {
            avatar.destroy();
            this.avatars.delete(id);
            this.eventBus.emit('avatar:removed', id);
        }
    }
}

class Avatar {
    constructor(id, config) {
        this.id = id;
        this.config = config;
        this.entity = null;
        this.controller = null;
        this.animations = new Map();
    }
    
    async load() {
        // 載入 VRM 模型
        if (this.config.vrmUrl) {
            this.entity = await this.loadVRM(this.config.vrmUrl);
        }
        
        // 設置控制器
        if (this.config.controller) {
            this.controller = new AvatarController(this.entity, this.config.controller);
        }
    }
    
    async loadVRM(url) {
        // VRM 載入邏輯
        // 返回 PlayCanvas Entity
    }
    
    playAnimation(name, options = {}) {
        const animation = this.animations.get(name);
        if (animation) {
            // 播放動畫
        }
    }
}

// ===== Network 模組 =====

class NetworkModule extends BaseModule {
    constructor(framework) {
        super(framework);
        this.connections = new Map();
        this.isHost = false;
        this.room = null;
    }
    
    async setup() {
        // 初始化網路連線
    }
    
    async createRoom(roomConfig) {
        this.room = new Room(roomConfig);
        this.isHost = true;
        
        // 設置主機邏輯
        this.setupHostHandlers();
        
        this.eventBus.emit('network:room_created', this.room);
        return this.room;
    }
    
    async joinRoom(roomId, userConfig) {
        this.room = await Room.join(roomId);
        
        // 設置客戶端邏輯
        this.setupClientHandlers();
        
        this.eventBus.emit('network:room_joined', this.room);
        return this.room;
    }
    
    setupHostHandlers() {
        // 處理玩家加入/離開
        // 同步狀態
    }
    
    setupClientHandlers() {
        // 接收狀態更新
        // 處理連線狀態
    }
    
    broadcast(event, data) {
        if (this.room) {
            this.room.broadcast(event, data);
        }
    }
}

// ===== Camera 模組 =====

class CameraModule extends BaseModule {
    constructor(framework) {
        super(framework);
        this.cameras = new Map();
        this.activeCamera = null;
        this.controls = null;
    }
    
    async setup() {
        // 創建預設相機
        this.createDefaultCamera();
    }
    
    createDefaultCamera() {
        const camera = this.framework.app.root.findByName('Camera');
        if (!camera) {
            const cameraEntity = new pc.Entity('Camera');
            cameraEntity.addComponent('camera');
            this.framework.app.root.addChild(cameraEntity);
        }
        
        this.activeCamera = camera;
        this.setupCameraControls();
    }
    
    setupCameraControls() {
        this.controls = new CameraController(this.activeCamera);
        
        // 設置軌道控制
        this.controls.setOrbitMode();
    }
    
    switchCamera(cameraId) {
        const camera = this.cameras.get(cameraId);
        if (camera) {
            this.activeCamera = camera;
            this.framework.app.scene.registry.camera = camera.camera;
        }
    }
}

// ===== 創作者 API 層 =====

class ContentCreator {
    constructor(canvasElement, options = {}) {
        this.framework = new ContentFramework(canvasElement, options);
        this.currentProject = null;
    }
    
    // 快速創建場景
    async createScene(config) {
        const scene = new Scene3D(config.name, this.framework);
        await scene.load(config);
        
        this.framework.scenes.set(config.name, scene);
        this.framework.currentScene = scene;
        
        return scene;
    }
    
    // 快速添加角色
    async addAvatar(config) {
        const avatarModule = this.framework.getModule('avatar');
        return await avatarModule.createAvatar(config.id, config);
    }
    
    // 快速設置網路房間
    async setupMultiplayer(config) {
        const networkModule = this.framework.getModule('network');
        
        if (config.isHost) {
            return await networkModule.createRoom(config);
        } else {
            return await networkModule.joinRoom(config.roomId, config.user);
        }
    }
    
    // 快速添加裝飾物件
    addDecoration(config) {
        const decorationModule = this.framework.getModule('decoration');
        return decorationModule.create(config);
    }
    
    // 事件監聽
    on(event, callback) {
        this.framework.eventBus.on(event, callback);
    }
    
    // 啟動應用
    start() {
        this.framework.app.start();
    }
}

// ===== 使用範例 =====

/*
// 創作者使用方式
const creator = new ContentCreator('#canvas');

// 創建場景
await creator.createScene({
    name: 'virtual-room',
    environment: {
        skybox: 'room-skybox',
        lighting: 'indoor'
    },
    assets: ['room-model', 'furniture-pack']
});

// 添加角色
const avatar = await creator.addAvatar({
    id: 'player1',
    vrmUrl: 'models/character.vrm',
    controller: {
        type: 'third-person',
        speed: 5
    }
});

// 設置多人連線
await creator.setupMultiplayer({
    isHost: true,
    maxPlayers: 10,
    roomName: 'My Room'
});

// 監聽事件
creator.on('avatar:created', (avatar) => {
    console.log('Avatar created:', avatar.id);
});

// 啟動
creator.start();
*/
```

### 關鍵設計原則

1. 簡化創作流程

```javascript 
// 一行代碼創建完整場景
await creator.createScene({
    template: 'meeting-room',  // 預設模板
    capacity: 20,
    features: ['avatar', 'chat', 'screen-share']
});
```

2. 模組化設計

每個功能都是獨立模組，可以按需載入：

- Avatar Module: VRM 支援、角色控制
- Network Module: 本地/遠程連線
- Camera Module: 多種相機模式
- UI Module: 聊天室、載入畫面
- Decoration Module: 物件裝飾系統

3. 配置驅動

```javascript
// 透過 JSON 配置快速建立內容
const sceneConfig = {
    name: "office-space",
    template: "office",
    avatars: {
        maxCount: 15,
        defaultModel: "business-avatar.vrm"
    },
    network: {
        type: "webrtc",
        features: ["voice", "text-chat"]
    },
    camera: {
        mode: "free-roam",
        constraints: { maxDistance: 50 }
    }
};
```

### 關鍵技術決策

1. 打包方式選擇

- 靜態打包：所有依賴在建置時確定，穩定但體積較大
- 動態載入：按需載入模組，靈活但複雜度高
- 建議：先從靜態打包開始，後續再考慮動態載入優化

2. API 設計原則

- 穩定的高階 API
- 避免：直接暴露內部 PlayCanvas API，太底層了容易變動

3. 相容性策略

- 使用 Adapter Pattern 包裝 PlayCanvas API
- 建立 Feature Detection 機制
- 提供優雅降級選項

開發工具鏈改善