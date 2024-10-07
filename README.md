# plug-in-system
研究插件系統

看到[此文章](https://ithelp.ithome.com.tw/articles/10345126)
有靈感請 Claude3.5協助產生以下範例

## 遊戲化範例

```js
// 角色狀態管理的核心類別
class CharacterContext {
  constructor() {
    this.state = {
      name: '',
      level: 1,
      hp: 100,
      mp: 50,
      skills: [],
      buffs: []
    };
    this.plugins = [];
  }

  use(plugin) {
    const context = {
      getState: () => this.state,
      setState: (newState) => {
        this.state = { ...this.state, ...newState };
      },
      // 提供修改特定屬性的輔助方法
      modifyHp: (amount) => {
        this.state.hp = Math.max(0, Math.min(100, this.state.hp + amount));
      },
      addSkill: (skill) => {
        if (!this.state.skills.includes(skill)) {
          this.state.skills.push(skill);
        }
      },
      addBuff: (buff) => {
        this.state.buffs.push({ ...buff, startTime: Date.now() });
      }
    };
    
    this.plugins.push(plugin(context));
  }

  getState() {
    return this.state;
  }
}

// 基礎角色設定插件
const baseCharacterPlugin = (context) => {
  context.setState({
    name: '勇者',
    hp: 100,
    mp: 50
  });
  
  return {
    name: 'baseCharacterPlugin',
    description: '設定角色基礎屬性'
  };
};

// 戰士職業插件
const warriorClassPlugin = (context) => {
  const currentState = context.getState();
  
  context.setState({
    hp: currentState.hp + 50, // 增加血量
    skills: [...currentState.skills, '橫掃千軍', '防禦姿態']
  });
  
  // 添加戰士的被動buff
  context.addBuff({
    name: '戰士之魂',
    effect: '物理防禦+20%',
    duration: 'passive'
  });
  
  return {
    name: 'warriorClassPlugin',
    description: '增加戰士職業特性'
  };
};

// 裝備插件
const equipmentPlugin = (context) => {
  const currentState = context.getState();
  
  context.setState({
    hp: currentState.hp + 20,
    mp: currentState.mp + 10
  });
  
  context.addBuff({
    name: '裝備加成',
    effect: '全屬性+10%',
    duration: 3600000 // 1小時
  });
  
  return {
    name: 'equipmentPlugin',
    description: '添加基礎裝備效果'
  };
};

// 使用示例
const character = new CharacterContext();
console.log('初始狀態:', character.getState());

// 依序加載插件
character.use(baseCharacterPlugin);
console.log('載入基礎屬性後:', character.getState());

character.use(warriorClassPlugin);
console.log('載入戰士職業後:', character.getState());

character.use(equipmentPlugin);
console.log('裝備基礎裝備後:', character.getState());
```

## 前端應用範例
```js
// 前端應用狀態管理的核心類別
class AppContext {
  constructor() {
    this.state = {
      user: null,
      theme: 'light',
      language: 'zh_TW',
      notifications: [],
      loading: false,
      error: null,
      features: new Set(),
      analytics: {
        events: []
      }
    };
    this.plugins = [];
  }

  use(plugin) {
    const context = {
      getState: () => this.state,
      setState: (newState) => {
        this.state = { ...this.state, ...newState };
      },
      // 提供常用的狀態操作方法
      setTheme: (theme) => {
        this.state.theme = theme;
        document.documentElement.setAttribute('data-theme', theme);
      },
      setLanguage: (lang) => {
        this.state.language = lang;
        document.documentElement.setAttribute('lang', lang);
      },
      addNotification: (notification) => {
        const newNotification = {
          id: Date.now(),
          timestamp: new Date(),
          ...notification
        };
        this.state.notifications = [...this.state.notifications, newNotification];
        return newNotification;
      },
      trackEvent: (event) => {
        const newEvent = {
          timestamp: new Date(),
          ...event
        };
        this.state.analytics.events = [...this.state.analytics.events, newEvent];
        return newEvent;
      }
    };
    
    const pluginInstance = plugin(context);
    this.plugins.push(pluginInstance);
    return pluginInstance;
  }

  getState() {
    return this.state;
  }

  // 將核心方法也暴露在實例上
  addNotification(notification) {
    const newNotification = {
      id: Date.now(),
      timestamp: new Date(),
      ...notification
    };
    this.state.notifications = [...this.state.notifications, newNotification];
    return newNotification;
  }

  trackEvent(event) {
    const newEvent = {
      timestamp: new Date(),
      ...event
    };
    this.state.analytics.events = [...this.state.analytics.events, newEvent];
    return newEvent;
  }
}

// 用戶認證插件
const authPlugin = (context) => {
  // 模擬從 localStorage 讀取用戶資訊
  const savedUser = localStorage.getItem('user');
  if (savedUser) {
    context.setState({ user: JSON.parse(savedUser) });
  }

  return {
    name: 'authPlugin',
    initialize: () => {
      // 監聽登入狀態變化
      window.addEventListener('storage', (e) => {
        if (e.key === 'user') {
          context.setState({ user: JSON.parse(e.newValue) });
        }
      });
    },
    login: async (credentials) => {
      context.setState({ loading: true });
      try {
        // 模擬 API 請求
        const response = await fetch('/api/login', {
          method: 'POST',
          body: JSON.stringify(credentials)
        });
        const user = await response.json();
        
        localStorage.setItem('user', JSON.stringify(user));
        context.setState({ user, loading: false });
        context.trackEvent({ type: 'LOGIN_SUCCESS' });
        context.addNotification({
          type: 'success',
          message: 'Successfully logged in'
        });
      } catch (error) {
        context.setState({ error: error.message, loading: false });
        context.trackEvent({ type: 'LOGIN_ERROR', error: error.message });
        context.addNotification({
          type: 'error',
          message: `Login failed: ${error.message}`
        });
      }
    },
    logout: () => {
      localStorage.removeItem('user');
      context.setState({ user: null });
      context.trackEvent({ type: 'LOGOUT' });
      context.addNotification({
        type: 'info',
        message: 'You have been logged out'
      });
    }
  };
};

// 主題管理插件
const themePlugin = (context) => {
  // 檢測系統主題偏好
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)');
  
  const updateTheme = (isDark) => {
    const theme = isDark ? 'dark' : 'light';
    context.setTheme(theme);
  };

  // 初始化主題
  updateTheme(prefersDark.matches);
  
  return {
    name: 'themePlugin',
    initialize: () => {
      // 監聽系統主題變化
      prefersDark.addEventListener('change', (e) => updateTheme(e.matches));
    },
    toggleTheme: () => {
      const currentState = context.getState();
      const newTheme = currentState.theme === 'light' ? 'dark' : 'light';
      context.setTheme(newTheme);
      context.trackEvent({ 
        type: 'THEME_CHANGE', 
        theme: newTheme 
      });
      context.addNotification({
        type: 'info',
        message: `Theme changed to ${newTheme}`
      });
    }
  };
};

// 多語言支援插件
const i18nPlugin = (context) => {
  const translations = {
    zh_TW: {
      welcome: '歡迎',
      settings: '設定'
      // ... 更多翻譯
    },
    en_US: {
      welcome: 'Welcome',
      settings: 'Settings'
      // ... 更多翻譯
    }
  };

  return {
    name: 'i18nPlugin',
    translate: (key) => {
      const { language } = context.getState();
      return translations[language]?.[key] || key;
    },
    setLanguage: (lang) => {
      if (translations[lang]) {
        context.setLanguage(lang);
        context.trackEvent({ 
          type: 'LANGUAGE_CHANGE', 
          language: lang 
        });
        context.addNotification({
          type: 'info',
          message: `Language changed to ${lang}`
        });
      }
    }
  };
};

// 使用示例
const app = new AppContext();

// 初始化各插件
const auth = app.use(authPlugin);
const theme = app.use(themePlugin);
const i18n = app.use(i18nPlugin);

// 插件使用示例
console.log('初始狀態:', app.getState());

// 模擬用戶登入
auth.login({
  username: 'user@example.com',
  password: 'password'
}).then(() => {
  console.log('登入後狀態:', app.getState());
});

// 切換主題
theme.toggleTheme();
console.log('切換主題後:', app.getState());

// 切換語言
i18n.setLanguage('en_US');
console.log('切換語言後:', app.getState());

// 正確的添加通知方式
app.addNotification({
  type: 'success',
  message: i18n.translate('welcome')
});
```

## 總結

這個插件系統的概念主要是透過 **「注入上下文（context）」** 的方式來實現靈活擴展功能。簡單來說，它允許外部插件動態地修改或影響系統的狀態，並且插件之間可以共用某些上下文資訊來實現功能。

### 核心概念解析：

1. **`Context` 類別**：
   - `Context` 類別負責管理系統中的狀態與插件。其主要目的是提供一個上下文環境，讓外部插件能夠存取和修改該環境的狀態（`current`）。
   - 透過 `use()` 方法，`Context` 可以接受一個插件，並將上下文環境提供給該插件進行操作。

2. **上下文（context）**：
   - 在 `use(plugin)` 方法中，`context` 是一個簡單的物件，包含兩個方法：
     - `getState()`: 取得當前的狀態（`current`）。
     - `setState(value)`: 設定新的狀態，並更新 `current` 的值。
   - 插件可以透過這些方法來讀取或改變系統的狀態。

3. **插件（plugin）**：
   - 插件是一個函數，它接收上下文（`context`），並利用這個上下文來進行某些操作。例如在這個例子中，插件會檢查當前狀態是否等於某個特定的 `value`，如果不是，它就會更新狀態。
   - 插件的特點是「動態」和「可插拔」，這意味著可以在不修改核心系統的情況下，通過外部插件來擴展系統的功能。

4. **狀態管理**：
   - 插件系統的核心是狀態管理，通過 `context.getState()` 和 `context.setState()` 來管理 `current` 的值。這使得插件能夠動態地改變系統的狀態，而這些變更可以持續存在於上下文中，供其他插件或系統的其他部分使用。

### 插件系統的運作流程：

1. **初始化**：
   - 系統初始化時，`Context` 物件會被建立，並且 `current` 狀態設為 `undefined`。

2. **插件注入**：
   - 當系統執行 `context.use(plugin)` 時，該插件會接收到當前的上下文（包含狀態的 `get` 和 `set` 方法），然後它可以根據狀態做出決定，比如設置新的狀態。

3. **狀態改變**：
   - 插件內部的邏輯會依據需求修改 `context` 的狀態。例如在這個例子中，插件會將狀態設為 `value`，如果當前狀態不是這個 `value`。

4. **擴展功能**：
   - 透過這種插件機制，系統可以不斷地添加不同的插件來實現額外的功能，且每個插件只需關心上下文提供的方法與狀態，而不需要了解整個系統的內部運作。

### 優點：

- **靈活性**：插件能夠動態地注入功能，而不需要修改原有的程式碼。這意味著你可以方便地擴展系統，而不破壞核心邏輯。
- **可重用性**：插件可以在不同的系統或情境中重用，因為它們只依賴於提供的上下文（如 `getState()` 和 `setState()`）。
- **低耦合**：插件和系統之間的耦合度很低，因為它們透過標準化的接口（上下文）來互動。

這樣的插件系統模式非常適合用於建構一個模組化、可擴展的應用程式。