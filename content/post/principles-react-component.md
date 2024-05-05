+++
title = "元件開發原則: React 基礎規範"
description = ""
date = 2023-11-05 21:30:00
author = "Rhan0"
tags=["principles"]
weight= 2
+++

## component

### 保持頂層 (Top-Level) 宣告

```jsx
// ❌
const App = () => {
  const Container = ({ children }) => <section>{children}</section>;
  const Button = ({ onClick }) => <button onClick={onClick}>Click Me!</button>;

  const handleClick = () => {
    // do something...
  };

  return (
    <Container>
      <Button onClick={handleClick} />
    </Container>
  );
};

// ✅
const Container = ({ children }) => <section>{children}</section>;
const Button = ({ onClick }) => <button onClick={onClick}>Click Me!</button>;

const App = () => {
  const handleClick = () => {
    // do something...
  };

  return (
    <Container>
      <Button onClick={handleClick} />
    </Container>
  );
};
```


### 盡量描述 UI 的形狀，Component 化

```jsx
const [HOME, PROFILE, SETTINGS] = ['home', 'profile', 'settings'];
const pages = {
  [HOME]: () => <Home />,
  [PROFILE]: () => <Profile />,
  [SETTINGS]: () => <Settings />,
};

// ❌
const App = () => {
  const history = useHistory();
  const { pathname } = useLocation();

  const routeToMenu = (type) => () => {
    history.push(`/${type}`);
  };

  const renderContent = () => {
    return pages[pathname]();
  };

  return (
    <section>
      <div>
        <button>Sign In</button>
        <button>Sign Up</button>
        <button>Forget Password</button>
      </div>
      <div>
        <div onClick={routeToMenu(HOME)}></div>
        <div onClick={routeToMenu(PROFILE)}></div>
        <div onClick={routeToMenu(SETTINGS)}></div>
      </div>
      <div>{renderContent()}</div>
    </section>
  );
};

// ✅ (描述 UI 的形狀，讓開發人員第一眼就可以看得出藍圖)
// 一個 Component file 盡量保持一個 Component 就好
// App.jsx
const App = () => {
  return (
    <Container>
      <Toolbar />
      <Menus />
      <Page />
    </Container>
  );
};

// Container.jsx
const Container = ({ children }) => <section>{children}</section>;

// Toolbar.jsx
const Toolbar = () => (
  <div>
    <button>Sign In</button>
    <button>Sign Up</button>
    <button>Forget Password</button>
  </div>
);

// Menus.jsx
const Menus = () => {
  const history = useHistory();

  const routeToMenu = (type) => () => {
    history.push(`/${type}`);
  };

  return (
    <div>
      <div onClick={routeToMenu(HOME)}></div>
      <div onClick={routeToMenu(PROFILE)}></div>
      <div onClick={routeToMenu(SETTINGS)}></div>
    </div>
  );
};

// Page.jsx
const Page = () => {
  const { pathname } = useLocation();

  const renderContent = () => {
    return pages[pathname]();
  };

  return <div>{renderContent()}</div>;
};
```


### 使用到 forwardRef 得描述 displayName

(否則 React Component DevTools 會抓不到該 Component 名稱，難以 Debug)

```jsx
// ❌
const Button = forwardRef(({ onClick }, ref) => (
  <button ref={ref} onClick={onClick}>
    Click Me!
  </button>
));

// ✅
const Button = forwardRef(({ onClick }, ref) => (
  <button ref={ref} onClick={onClick}>
    Click Me!
  </button>
));

Button.displayName = 'Button';
```


### 請勿違反設計原則

```jsx
// ❌
class Content extends Component {
  constructor(props) {
    super(props);
    this.state = {
      open: false,
    };
  }

	// eslint 會提示 unused function，此時拔除會壞掉
  openDialog = () => this.setState({ open: true });

  render() {
    return (
      <Fragment>
        <Title>content-title</Title>
        <Dialog open={this.state.open}>
          <DialogTitle>dialog-title</DialogTitle>
          <DialogContent>dialog-content</DialogContent>
          <DialogFooter>dialog-footer</DialogFooter>
        </Dialog>
      </Fragment>
    );
  }
}

class Page extends Component {
  constructor(props) {
    super(props);
    this.dialogRef = createRef();
  }

  handleOpenDialog = () => {
    this.dialogRef.openDialog();
  };

  render() {
    return (
      <div>
        <Content ref={this.dialogRef} />
        <button onClick={this.handleOpenDialog}>open dialog</button>
      </div>
    );
  }
}

// ✅
class Content extends Component {
  render() {
    return (
      <Fragment>
        <Title>content-title</Title>
        <Dialog open={this.props.dialogOpen}>
          <DialogTitle>dialog-title</DialogTitle>
          <DialogContent>dialog-content</DialogContent>
          <DialogFooter>dialog-footer</DialogFooter>
        </Dialog>
      </Fragment>
    );
  }
}

class Page extends Component {
  constructor(props) {
    super(props);
    this.state = {
      dialogOpen: false,
    };
  }

  handleOpenDialog = () => {
    this.setState({ dialogOpen: true });
  };

  render() {
    return (
      <div>
        <Content dialogOpen={this.dialogOpen} />
        <button onClick={this.handleOpenDialog}>open dialog</button>
      </div>
    );
  }
}
```


### 少使用 lodash，以原生 api 為主

```jsx
const value = {
  0: 123,
  1: 456,
  2: 789,
};

// ❌ 使用 lodash reduce，無法第一眼就看出 value 是哪種資料型態
const sum = _.reduce(value, (acc, cur) => acc + cur, 0);

// ✅ 使用 ES6 Object.values，即可得知 value 為物件型態
const sum = Object.values(value).reduce((acc, cur) => acc + cur, 0);
```


### props default value 設置

```jsx
const InputLabel = ({ id, value }) => (
  <div>
    <label htmlFor={id}>{id}</label>
    <input type="text" value={value} readOnly />
  </div>
);

// ❌ class component 若要給定 props default value
// 需綁定在 defaultProps 上
class Contact extends Component {
  render() {
    const { profile = {} } = this.props;
    const { id = '', name = '', address = '' } = profile;
    return (
      <div>
        <InputLabel id="id" value={id} />
        <InputLabel id="name" value={name} />
        <InputLabel id="address" value={address} />
      </div>
    );
  }
}

// ✅
class Contact extends Component {
  render() {
    const {
      profile: { id, name, address },
    } = this.props;
    return (
      <div>
        <InputLabel id="id" value={id} />
        <InputLabel id="name" value={name} />
        <InputLabel id="address" value={address} />
      </div>
    );
  }
}

Contact.propTypes = {
  profile: PropTypes.shape({
    id: PropTypes.string,
    name: PropTypes.string,
    address: PropTypes.string,
  }),
};

Contact.defaultProps = {
  profile: {
    id: '',
    name: '',
    address: '',
  },
};

// ✅ function component 若要給定 props default value
// 直接宣告在上方即可
const Contact = ({ profile: { id = '', name = '', address = '' } = {} }) => (
  <div>
    <InputLabel id="id" value={id} />
    <InputLabel id="name" value={name} />
    <InputLabel id="address" value={address} />
  </div>
);

Contact.propTypes = {
  profile: PropTypes.shape({
    id: PropTypes.string,
    name: PropTypes.string,
    address: PropTypes.string,
  }),
};
```


### callback function 描述

```jsx
// ❌ 若是多行的 callback function，則需另外定義變數描述此行為
// 否則無法第一眼看出此 Component 的行為
const ButtonGroups = ({ onCancel }) => {
  return (
    <div>
      <Button
        onClick={() => {
          fetch('https://gorilla-sc/create')
            .then((res) => res.json())
            .then(() => {
              alert('successfully created !!');
            });
        }}
      >
        Create
      </Button>
      <Button
        onClick={() => {
          fetch('https://gorilla-sc/read')
            .then((res) => res.json())
            .then((data) => {
              console.log('read data: ', data);
            });
        }}
      >
        Read
      </Button>
      <Button onClick={() => onCancel()}>Cancel</Button>
    </div>
  );
};

// ✅ 描述該 callback function 行為，使 Component 更容易閱讀
// 若是接過來的 props callback function 則以 "on" 為字首描述
const ButtonGroups = ({ onCancel }) => {
  // 若 callback function 權責在 parent 手上，則以 "handle" 為字首描述
  const handleCreate = () => {
    fetch('https://gorilla-sc/create')
      .then((res) => res.json())
      .then(() => {
        alert('successfully created !!');
      });
  };

  const handleRead = () => {
    fetch('https://gorilla-sc/read')
      .then((res) => res.json())
      .then((data) => {
        console.log('read data: ', data);
      });
  };

  return (
    <div>
      <Button onClick={handleCreate}>Create</Button>
      <Button onClick={handleRead}>Read</Button>
      {/** callback function 只有一行，則可不需要再加以描述，因為「第一眼」看得出來 */}
      {/** 按鈕 => onClick => 觸發 onCancel function */}
      <Button onClick={() => onCancel()}>Cancel</Button>
    </div>
  );
};
```


## useState

### state 以及 setState 命名必須配對

```jsx
✅ const [filterText, setFilterText] = useState('')
❌ const [filterText, setText] = useState('')
```


### 若有搭配條件式判定，則 state 描述須確實 (verb. + adj.)

```jsx
✅ const [isWaiting, setIsWaiting] = useState(false);
❌ const [waiting, setWaiting] = useState(false);

✅ if (isWaiting) { // do something } => 若等待中，則 ... => 語意正確
❌ if (waiting) { // do something } => 若等待，則 ... => 語意不正確
```


### 勿過度裝載 props 當作 state，這會讓 Component 變複雜

```jsx
// ✅
const FilterableProductTable = ({ products }) => (
	<div>
    <SearchBar />
    <ProductTable products={products} />
  </div>
);

// ❌
const FilterableProductTable = ({ products }) => {
  const [currentProduct, setCurrentProduct] = useState(products);
  return (
    <div>
      <SearchBar />
      <ProductTable products={currentProduct} />
    </div>
  )
}; 
```


### 若為 Object 或 Array，則必須遵照 immutable 規範

```jsx
// state
const [contactInfos, setContactInfos] = useState({
	name: '',
  email: '',
	address: '',
})

const handleUpdateContactInfos = (e) => {
  // ❌
  contactInfos[e.target.name] = e.target.value;
  setContactInfos(contactInfos);

  // ✅ (使用 ES6 spread operator 以 shallow copy)
	setContactInfos(prev => ({
     ...prev,
     [e.target.name]: e.target.value,
  });

  // ✅ (使用 immer library，則須保證此值不會被其他地方 mutable 到，
  // 否則可能會壞掉 (Auto Freeze))
  setContactInfos(prev => {
    const newContactInfos = produce(prev, draft => {
      draft[e.target.name] = e.target.value,
    });
    return newContactInfos;
  })
};


// props
// ❌ 這樣會改變到 data 這個物件的源頭
const App = ({ data }) => {
  data.person = 'tom';
  data.phone = '0912355678';

  return <Contact {...data} />;
};

// ✅ 使用 ES6 spread 攤平並直接定義在 props 上
const App = ({ data }) => (
  <Contact  
    {...data}
    person="tom"
    phone="0912355678"
  />
);

// ✅ 使用 ES6 spread 做 shallow copy 宣告新的 props
const App = ({ data }) => {
  const newData = {
    ...data,
    person: 'tom',
    phone: '0912355678',
  };

  return <Contact {...newData} />;
};

// ✅ 使用 immer library 宣告新的 props
const App = ({ data }) => {
  const newData = produce(data, draft => {
    draft.person = 'tom'
    draft.phone = '0912355678'
  })
  
  return <Contact {...newData} />;
};
```


### Lift State Up or Down / Move Content Up

```jsx
const ExpensiveTree = () => {
  const now = performance.now();

  while (performance.now() - now < 100) {
    // 故意延遲，不做任何事情而等待 100ms
  }

  return <p>我是一個非常慢的 component tree</p>;
};

// ❌ setColor 後的 render 影響到 ExpensiveTree 造成無謂的渲染
const App = () => {
  const [color, setColor] = useState('red');

  return (
    <div>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
};

// ✅ 方法一：把 state 往下搬移，即可解決多於渲染問題
// (因為往下搬移後，只有 InputParagraph 這個 Component 會 render)
const InputParagraph = () => {
  const [color, setColor] = useState('red');

  return (
    <>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
    </>
  );
};

const App = () => {
  return (
    <div>
      <InputParagraph />
      <ExpensiveTree />
    </div>
  );
};

// ✅ 方法二：把 ExpensiveTree 變成 children
// 對 React 而言，children 的異動是它偵測不到的
// 所以 ExpensiveTree 只 render 一次，並且還能避掉之後 setColor 導致的 render 行為
const Layout = ({ children }) => {
  const [color, setColor] = useState('red');

  return (
    <div>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
      {children}
    </div>
  );
};

const App = () => {
  return (
    <Layout>
      <ExpensiveTree />
    </Layout>
  );
};
```


## useEffect

### 有時 effect 是不必要的

```jsx
// ❌
const App = () => {
  const [data, setData] = useState(null);
  const [loadKey, setLoadKey] = useState(null);

  useEffect(() => {
    if (loadKey) {
      fetch('https://www.test.com')
        .then((res) => res.json())
        .then((data) => setData(data));
    }
  }, [loadKey]);

  const handleGetData = () => {
    setLoadKey(Math.random());
  };

  return (
    <div>
      <button onClick={handleGetData}>get data</button>
      {data}
    </div>
  );
};

// ✅
const App = () => {
  const [data, setData] = useState(null);

  const handleGetData = () => {
    fetch('https://www.test.com')
      .then((res) => res.json())
      .then((data) => setData(data));
  };

  return (
    <div>
      <button onClick={handleGetData}>get data</button>
      {data}
    </div>
  );
};
```


### dependencies 越少越好

(當一個  effect dependencies 很多的時候，很容易造成閱讀上的困難)

```jsx
// ❌
useEffect(() => {
  const checkIdurl = `https://www.movie-ticket.com/id=${id}`;

  fetch(checkIdurl)
    .then((res) => res.json())
    .then((data) => {
      if (!data.isLegal) alert('identified error');
      else {
        const orderTicketUrl = `https://www.movie-ticket.com/id=${id}&name=${name}&date=${date}&people=${people}&theatre=${theatre}`;

        fetch(orderTicketUrl, { method: 'POST' })
          .then((res) => res.json())
          .then((data) => {
            if (data.success) alert('order successfully');
            else alert('order failed');
          });
      }
    });
}, [id, name, date, people, theatre, movieId]);

// ✅ (可透過 useCallback 分離關注點並加以描述，使 effect 更容易閱讀，各司其職)
const orderTicket = useCallback(() => {
  const url = `https://www.movie-ticket.com/id=${id}&movieId=${movieId}&date=${date}&people=${people}&theatre=${theatre}`;

  fetch(url, { method: 'POST' })
    .then((res) => res.json())
    .then((data) => {
      if (data.success) alert('order successfully');
      else alert('order failed');
    });
}, [id, movieId, date, people, theatre]);

const verifyIdToOrderTicket = useCallback(async () => {
  const url = `https://www.movie-ticket.com/id=${id}`;

  fetch(url, { method: 'POST' })
    .then((res) => res.json())
    .then((data) => {
      if (data.isLegal) orderTicket();
      else alert('identified error');
    });
}, [id, orderTicket]);

useEffect(() => {
  verifyIdToOrderTicket();
}, [verifyIdToOrderTicket]);

// ✅ (或至少針對 effect 內容加以描述，使其更容易閱讀)
useEffect(() => {
  const orderTicket = () => {
    const url = `https://www.movie-ticket.com/id=${id}&name=${name}&date=${date}&people=${people}&theatre=${theatre}`;

    fetch(url, { method: 'POST' })
      .then((res) => res.json())
      .then((data) => {
        if (data.success) alert('order successfully');
        else alert('order failed');
      });
  };

  const verifyIdToOrderTicket = () => {
    const url = `https://www.movie-ticket.com/id=${id}`;

    fetch(url, { method: 'POST' })
      .then((res) => res.json())
      .then((data) => {
        if (!data.isLegal) alert('identified error');
        else {
          orderTicket();
        }
      });
  };

  verifyIdToOrderTicket();
}, [id, name, date, people, theatre, movieId]);
```


### effect 內的 function 宣告盡量不要超過一層

```jsx
// ❌
useEffect(() => {
  const verifyIdToOrderTicket = () => {
    const verifyIdUrl = `https://www.movie-ticket.com/id=${id}`;

    fetch(verifyIdUrl)
      .then((res) => res.json())
      .then((data) => {
        if (!data.isLegal) alert('identified error');
        else {
          const orderTicket = () => {
            const orderTicketUrl = `https://www.movie-ticket.com/id=${id}&name=${name}&date=${date}&people=${people}&theatre=${theatre}`;

            fetch(orderTicketUrl, { method: 'POST' })
              .then((res) => res.json())
              .then((data) => {
                if (data.success) alert('order successfully');
                else alert('order failed');
              });
          };

          orderTicket();
        }
      });
  };

  verifyIdToOrderTicket();
}, [id, name, date, people, theatre, movieId]);

// ✅ (盡量保持 function 宣告在第一層就好)
useEffect(() => {
  const orderTicket = () => {
    const url = `https://www.movie-ticket.com/id=${id}&name=${name}&date=${date}&people=${people}&theatre=${theatre}`;

    fetch(url, { method: 'POST' })
      .then((res) => res.json())
      .then((data) => {
        if (data.success) alert('order successfully');
        else alert('order failed');
      });
  };

  const verifyIdToOrderTicket = () => {
    const url = `https://www.movie-ticket.com/id=${id}`;

    fetch(url)
      .then((res) => res.json())
      .then((data) => {
        if (!data.isLegal) alert('identified error');
        else {
          orderTicket();
        }
      });
  };

  verifyIdToOrderTicket();
}, [id, name, date, people, theatre, movieId]);
```


## useCallback (記憶「函式」) / useMemo (記憶「值」)

### 有需要使用再使用

```jsx
// ❌ 不需要無限擴充，只為了渲染一次
// 大部分渲染的成本，都比使用 useCallback 後的 Component 還要來得低
const App = () => {
  const handleClick = useCallback(() => {
    // do something ...
  }, []);

  const handleOpen = useCallback(() => {
    // do something ...
  }, []);

  const handleClose = useCallback(() => {
    // do something ...
  }, []);

  return (
    <Layout>
      <Button onClick={handleClick}>Click</Button>
      <Button onClick={handleOpen}>Open</Button>
      <Button onClick={handleClose}>Close</Button>
    </Layout>
  );
};

// ✅ 拔除掉 useCallback 後，整份程式碼變得很單純
const App = () => {
  const handleClick = () => {
    // do something ...
  };

  const handleOpen = () => {
    // do something ...
  };

  const handleClose = () => {
    // do something ...
  };

  return (
    <Layout>
      <Button onClick={handleClick}>Click</Button>
      <Button onClick={handleOpen}>Open</Button>
      <Button onClick={handleClose}>Close</Button>
    </Layout>
  );
};
```


### 使用時機

- 畫面確實有明顯效能問題
  - 肉眼可看出卡頓
  - 透過 console.time + console.timeEnd 量測且超過能接受的值 (ex. 1ms))

- 減少 useEffect dependencies 複雜度
    
```jsx
const [datas, setDatas] = useState([]);

useEffect(() => {
  fetch(`https://imdb/get-movie?id=${id}`)
    .then((res) => res.json())
    .then((json) => {
      if (json.success) {
        fetch(`https://imdb/get-movie?people=${people}&name=${name}&date=${date}`)
          .then((res) => res.json())
          .then((json) => {
            setDatas(json);

            fetch(`https://imbd/send-mail?mail=${mail}?content=${content}`)
              .then((res) => res.json())
              .then((json) => {
                if (json.success) {
                  alert('mail sent successfully !!');
                }
              });
          });
      }
    });
  // ❌ dependencies 過多，很難理解觸發條件為何
}, [id, name, date, people, mail, content]);
```
    
```jsx
const sendMail = useCallback(() => {
  fetch(`https://imbd/send-mail?mail=${mail}?content=${content}`)
    .then((res) => res.json())
    .then((json) => {
      if (json.success) {
        alert('mail sent successfully !!');
      }
    });
}, [mail, content]);

const getDatas = useCallback(() => {
  fetch(`https://imdb/get-movie?people=${people}&name=${name}&date=${date}`)
    .then((res) => res.json())
    .then((json) => {
      setDatas(json);
      sendMail();
    });
}, [name, date, people, sendMail]);

const checkId = useCallback(() => {
  fetch(`https://imdb/get-movie?id=${id}`)
    .then((res) => res.json())
    .then((json) => {
      if (json.success) {
        getDatas();
      }
    });
}, [id, getDatas]);

useEffect(() => {
  checkId();
  // ✅ dependencies 變簡單，可以直覺的指到該觸發條件
}, [checkId]);
```


- child component 有效能問題
  - 則丟給此 component 的 props 需用 useMemo / useCallback 包裹，且 child component 也需被 React.memo 包裹