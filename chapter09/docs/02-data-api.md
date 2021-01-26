# Ch09 - 공공 데이터를 이용한 웹 애플리케이션 개발

## axios로 API 호출해서 데이터 받아오기

create react-app으로 생성한 hello-react 프로젝트에 이어서 공공 데이터를 활용하여 지역 별로 미세먼지와 같은 대기오염 정보를 보여 줄 수 있도록 수정해보겠습니다. 

>hello-react 프로젝트에 사용되는 공공데이터는 https://data.go.kr/data/15073861/openapi.do 입니다. 
>
>요청 API주소 예 : http://apis.data.go.kr/B552584/ArpltnInforInqireSvc/getCtprvnRltmMesureDnsty?serviceKey={발급받은키값}&dataTerm=3MONTH&InformCode	=PM10&numOfRows=100&returnType=JSON&sidoName=경남
>
>

axios는 현재 가장 많이 사용되고 있는 자바스크립트 HTTP 클라이언트입니다. 이 라이브러리의 특징은 HTTP 요청을 Promise 기반으로 처리하고 있다는 점입니다. 

axios를 사용하기 위해  hello-react 프로젝트에 axios라이브러리를 설치해보겠습니다. 

```
cd hello-react
yarn add axios
```

axios를 사용하여 API를 호출해보기 위해 src/App.js 파일을 다음과 같이 수정합니다.

```javascript
import React, {useState} from 'react';
import { Route , Link, Switch } from 'react-router-dom';
import About from './About';
import Home from './Home';
import Profiles from './Profiles';
import HistorySample from './HistorySample';
import axios from 'axios';

const App = () => {

    const [data, setData] = useState(null);

    const onClick = async () => {
        try {
            const response = await axios.get(
                'http://localhost:3001/api',
            );
            setData(response.data);
        } catch (e) {
            console.log(e);
        }
    };
  
    return (
        <div>
            <ul>
                <li>
                    <Link to="/">홈</Link>
                </li>
                <li>
                <Link to="/about">소개</Link>
                </li>
                <li>
                    <Link to="/profiles">프로필</Link>
                </li>
                <li>
                    <Link to='/history'>History 예제</Link>
                </li>
            </ul>
            <hr/>
            <Switch>
            <Route path="/" component={Home} exact={true} />
            <Route path={["/about", "/info"]} component={About} />
            <Route path="/profiles" component={Profiles} />
            <Route path="/history" component={HistorySample} />
                <Route
                    // path를 따로 정의하지 않으면 모든 상황에 렌더링됨
                    render={({ location }) => (
                        <div>
                            <h2>이 페이지는 존재하지 않습니다:</h2>
                            <p>{location.pathname}</p>
                        </div>
                    )}
                />
            </Switch>

            <div>
                <div>
                    <button onClick={onClick}>불러오기</button>
                </div>
                {data && <textarea rows={7} value={JSON.stringify(data, null, 2)} readOnly={true} />}
            </div>

        </div>
);
};

export default App;
```

불러오기 버튼을 클릭하면 API를 호출하고 이에 대한 응답을 컴포넌트 상태에 넣어서 보여주는 예제입니다. 

> 컴포넌트 상태(state)?
>
> 컴포넌트 내부에서 변경될 수 있는 값을 의미, `useState함수`의 인자에는 상태의 초기값(숫자, 문자열, 객체, 배열)을 넣어줍니다.
>
> useState함수는 호출되면 배열을 반환하는데, 첫번째 원소는 현재 상태, 두번째 원소는 상태를 바꿔주는 함수입니다. 이 함수를 `세터함수`라고 부릅니다. 상태의 값을 변경하고자 하는 경우 세터함수를 사용해서 변경해주면 됩니다. 

불러오기 버튼 클릭으로, onClick 이벤트가 실행되면 onClick함수가 실행됩니다. onClick함수는  axios.get을 사용하여 파라미터로 전달된 주소에 GET요청을 해주고 이에 대한 결과를 세터함수를 통해 data 상태를 변경해줍니다. 



![img](https://dbcore-assets-public.s3.amazonaws.com/tutorials/tutorial-nodejs-web-development/ch02/images/ch09-react-api.png)

## UI 만들기

지역 별로 미세먼지와 같은 대기오염 정보를 보여주기 위해 필요한 UI컴포넌트를 생성해보겠습니다. 

컴포넌트 스타일은 styled-components를 사용하여 추가할 것입니다. 

> styled-components?
>
> 자바스크립트 파일안에 스타일을 선언하는 CSS-in-JS 라이브러리 중 하나

styled-components를 사용하기 위해  hello-react 프로젝트에 styled-components라이브러리를 설치해보겠습니다. 

```
cd hello-react
yarn add styled-components
```

그리고 src디렉터리 안에 components 디렉터리를 생성한다음, AirItem.js와 AirList.js 파일을 생성합니다.  AirItem은 각 대기오염 정보를 보여주는 컴포넌트이고,  AirList는 API를 요청하고 대기 오염 데이터가 들어있는 배열을 컴포넌트 배열로 변환하여 렌더링 해주는 컴포넌트입니다.

### AirItem

먼저, AirItem컴포넌트 코드를 작성해보겠습니다.  대기 오염 데이터에는 다양한 필드로 이루어져 있지만,  hello-react 프로젝트는 그 중 다음 필드를 리액트 컴포넌트에 나타내겠습니다.

*  stationName : 측정소명
*  dataTime : 측정일
* o3Value : 오존 농도
* pm10Value : 미세먼지(PM10)농도 

 AirItem.js를 다음과 같이 작성합니다. 

```javascript
import React from 'react';
import styled from 'styled-components';


const AirItemBlock = styled.div`
  display: flex;



.thumbnail {
    margin-right: 1rem;
    img {
      display: block;
      width: 160px;
      height: 100px;
      object-fit: cover;
    }
  }
  .contents {
    h2 {
      margin: 0;
      a {
        color: black;
      }
    }
    p {
      margin: 0;
      line-height: 1.5;
      margin-top: 0.5rem;
      white-space: normal;
    }
  }
  & + & {
    margin-top: 3rem;
  }
`;
const AirItem = ({ article }) => {
    const {  stationName, dataTime, o3Value, pm10Value } = article;
    return (
        <AirItemBlock>
            <div className="contents">
            <h2>
                {stationName}
            </h2>
                <p>측정일 : {dataTime}</p>
                <p>오존 농도 : {o3Value}</p>
                <p>미세먼지(PM10)농도 : {pm10Value}</p>
        </div>
        </AirItemBlock>
);
};


export default AirItem;
```

AirItem컴포넌트는 article 이라는 객체를 props로 통으로 받아와서 렌더링해주게 됩니다.  styled-components라이브러리의 `styled.태그명`을 사용하여 스타일링이 적용된 컴포넌트를 상단에 생성한다음 적용하고 싶은 엘리먼트를 감싸줌으로써 컴포넌트에 스타일을 적용하고 있습니다. 



### AirList

다음으로 AirList컴포넌트를 만들어보겠습니다. 나중에 이 컴포넌트에서 API 요청하게 되지만 현재는 sampleArticle 객체에 미리 예시 데이터를 넣은 후 가짜 데이터가 출력되도록 AirList.js에 코드를 작성해보겠습니다.  

```javascript
import React from 'react';
import styled from 'styled-components';
import AirItem from './AirItem';


const AirListBlock = styled.div`
  box-sizing: border-box;
  padding-bottom: 3rem;
  width: 768px;
  margin: 0 auto;
  margin-top: 2rem;
  @media screen and (max-width: 768px) {
    width: 100%;
    padding-left: 1rem;
    padding-right: 1rem;
  }
`;



const sampleArticle = {
    "so2Grade": "1",
    "coFlag": null,
    "khaiValue": "160",
    "so2Value": "0.008",
    "coValue": "0.4",
    "pm10Flag": null,
    "o3Grade": "2",
    "pm10Value": "87",
    "khaiGrade": "3",
    "sidoName": "경남",
    "no2Flag": null,
    "no2Grade": "1",
    "o3Flag": null,
    "so2Flag": null,
    "dataTime": "2021-01-15 14:00",
    "coGrade": "1",
    "no2Value": "0.027",
    "stationName": "진영읍",
    "pm10Grade": "3",
    "o3Value": "0.040"
};


const AirList = () => {
            return (
            <AirListBlock>
            <AirItem article={sampleArticle} />
            <AirItem article={sampleArticle} />
            <AirItem article={sampleArticle} />
            <AirItem article={sampleArticle} />
            <AirItem article={sampleArticle} />
            <AirItem article={sampleArticle} />
            </AirListBlock>
            );
        };



export default AirList;
```

AirList컴포넌트를 렌더링해보기 위해 src디렉터리의 App.js 파일을 다음과 같이 수정합니다. 

Api 예제를 위한 Link컴포넌트와, Route컴포넌트가 추가되었습니다. 

```javascript
import React, {useState} from 'react';
import { Route , Link, Switch } from 'react-router-dom';
import About from './About';
import Home from './Home';
import Profiles from './Profiles';
import HistorySample from './HistorySample';
import axios from 'axios';
import AirList from './components/AirList';

const App = () => {


    const [data, setData] = useState(null);

    const onClick = async () => {
        try {
            const response = await axios.get(
                'http://localhost:3001/api?sidoName=경남',
            );
            setData(response.data);
        } catch (e) {
            console.log(e);
        }
    };


    return (
        <div>
            <ul>
                <li>
                    <Link to="/">홈</Link>
                </li>
                <li>
                <Link to="/about">소개</Link>
                </li>
                <li>
                    <Link to="/profiles">프로필</Link>
                </li>
                <li>
                    <Link to='/history'>History 예제</Link>
                </li>
                <li>
                    <Link to='/api'>Api 예제</Link>
                </li>
            </ul>
            <hr/>
            <Switch>
            <Route path="/" component={Home} exact={true} />
            <Route path={["/about", "/info"]} component={About} />
            <Route path="/profiles" component={Profiles} />
            <Route path="/history" component={HistorySample} />
            <Route path="/api" component={AirList} />
                <Route
                    // path를 따로 정의하지 않으면 모든 상황에 렌더링됨
                    render={({ location }) => (
                        <div>
                            <h2>이 페이지는 존재하지 않습니다:</h2>
                            <p>{location.pathname}</p>
                        </div>
                    )}
                />
            </Switch>

            <div>
                <div>
                    <button onClick={onClick}>불러오기</button>
                </div>
                {data && <textarea rows={7} value={JSON.stringify(data, null, 2)} readOnly={true} />}
            </div>


        </div>
);
};

export default App;
```

브라우저에서 http://localhost:3000/api 으로 접속하게 되면 다음과 같이 sampleArticle가 여러번 출력되는 것을 확인할 수 있습니다. 

![img](https://dbcore-assets-public.s3.amazonaws.com/tutorials/tutorial-nodejs-web-development/ch02/images/ch09-api-list.png)



## 데이터 연동하기

이제 실질적으로 AirList컴포넌트에 API를 호출해 보겠습니다. 컴포넌트가 화면에 보이는 시점에 API를 요청하기 위해 `useEffect`를 사용합니다.  useEffect는 컴포넌트가 처음 렌더링 되는 시점에 실행되기 때문입니다. 

 components/AirList.js 파일을 다음과 같이 수정합니다. 

```javascript
import React, {useState, useEffect} from 'react';
import styled from 'styled-components';
import AirItem from './AirItem';
import axios from 'axios';

const AirListBlock = styled.div`
  box-sizing: border-box;
  padding-bottom: 3rem;
  width: 768px;
  margin: 0 auto;
  margin-top: 2rem;
  @media screen and (max-width: 768px) {
    width: 100%;
    padding-left: 1rem;
    padding-right: 1rem;
  }
`;


/*
const sampleArticle = {
    "so2Grade": "1",
    "coFlag": null,
    "khaiValue": "160",
    "so2Value": "0.008",
    "coValue": "0.4",
    "pm10Flag": null,
    "o3Grade": "2",
    "pm10Value": "87",
    "khaiGrade": "3",
    "sidoName": "경남",
    "no2Flag": null,
    "no2Grade": "1",
    "o3Flag": null,
    "so2Flag": null,
    "dataTime": "2021-01-15 14:00",
    "coGrade": "1",
    "no2Value": "0.027",
    "stationName": "진영읍",
    "pm10Grade": "3",
    "o3Value": "0.040"
};
*/

const AirList = () => {

    const [articles, setArticles] = useState(null);
    const [loading, setLoading] = useState(false);

        useEffect(() => {
            // async를 사용하는 함수 따로 선언
            const fetchData = async () => {
                setLoading(true);
                try {
                    const response = await axios.get(
                    'http://localhost:3001/api?sidoName=경남',
                        );
                    setArticles(response.data.response.body.items);
                    //console.log(response.data.response.body.items)
                } catch (e) {
                    console.log(e);
                }
                setLoading(false);
            };
            fetchData();
        }, []);

        // 대기 중일 때
        if (loading) {
            return <AirListBlock>대기 중…</AirListBlock>;
        }
        // 아직 articles 값이 설정되지 않았을 때
        if (!articles) {
            return null;
        }
        // articles 값이 유효할 때
        return (
            <AirListBlock>
                {articles.map(article => (
                    <AirItem key={article.stationName} article={article} />
                ))}
            </AirListBlock>
        );
};



export default AirList;
```

컴포넌트가 처음 렌더링 되는 시점에 useEffect으로 axios.get을 사용해 API를 호출합니다. 결과값을 세터함수를 사용하여 articles 상태를 업데이트합니다.  추가로 loading이라는 상태도 관리하여 API요청이 대기 중인지 판별합니다. 요청이 대기 중일때는 loading값이 true가 되고 요청이 끝나면 loading값이 false가 됩니다.  

데이터를 불러와서 대기오염 정보 배열을 map 함수를 사용하여 컴포넌트 배열로 변환하는 과정에서 주의해야 하는 부분이 있습니다. map 함수를 사용하기 전에 해당 값이 현재 null인지 아닌지 검사해야 합니다. 만약 데이터가 없을때 map 함수를 사용하게 되면 렌더링 과정에서 오류가 발생하게 됩니다. 

key값을 설정해주지않으면 에러가 표시되기 때문에, 데이터가 가진 고유값으로 설정해줍니다. 고유값이 없다면 map 함수에 전달되는 콜백 함수의 인수인 index를 사용해도 됩니다. 

브라우저에서 http://localhost:3000/api 으로 다시 접속해보겠습니다. 

여러 지역별로 측정일, 오존 농도, 미세먼지 농도 등이 출력되는 것을 확인 할 수 있습니다. 

![img](https://dbcore-assets-public.s3.amazonaws.com/tutorials/tutorial-nodejs-web-development/ch02/images/ch09-api-list-update.png)

## 카테고리 기능 구현하기

지금까지 경남 지역의 동별로 대기오염 정보를 렌더링해주었습니다. 

이번에는 시도 이름(전국, 서울, 부산, 대구, 인천, 광주, 대전, 울산, 경기, 강원, 충북, 충남, 전북, 전남, 경북, 경남, 제주, 세종) 을 선택할 수 있는 기능을 구현해보겠습니다. 

먼저 카테고리를 선택할 수 있는 UI를 생성합니다.  components 디렉터리에 Categories.js 파일을 다음과 같이 생성합니다. 

```javascript
import React from 'react';
import styled from 'styled-components';
import {NavLink} from 'react-router-dom';

const categories = [
    {
        sidoName: '전국',
    },
    {
        sidoName: '서울',
    },
    {
        sidoName: '부산',
    },
    {
        sidoName: '대구',
    },
    {
        sidoName: '인천',
    },
    {
        sidoName: '광주',
    },
    {
        sidoName: '대전',
    },
    {
        sidoName: '울산',
    },
    {
        sidoName: '경기',
    },
    {
        sidoName: '강원',
    },
    {
        sidoName: '충북',
    },
    {
        sidoName: '충남',
    },
    {
        sidoName: '전북',
    },{
        sidoName: '전남',
    },
    {
        sidoName: '경북',
    },
    {
        sidoName: '경남',
    },
    {
        sidoName: '제주',
    },
    {
        sidoName: '세종',
    },
];

const CategoriesBlock = styled.div`
  display: flex;
  padding: 1rem;
  width: 768px;
  margin: 0 auto;
  @media screen and (max-width: 768px) {
    width: 100%;
    overflow-x: auto;
  }
`;

const Category = styled(NavLink)`
  font-size: 1.125rem;
  cursor: pointer;
  white-space: pre;
  text-decoration: none;
  color: inherit;
  padding-bottom: 0.25rem;
  &:hover {
    color: #495057;
  }
  &.active {
    font-weight: 600;
    border-bottom: 2px solid #22b8cf;
    color: #22b8cf;
    &:hover {
      color: #3bc9db;
    }
  }
  & + & {
    margin-left: 1rem;
  }
`;
const Categories = () => {
    return (
        <CategoriesBlock>
            {categories.map(c => (
                <Category
                          key={c.sidoName}
                          activeClassName="active" //NavLink가 활성화 되면 적용되는 className
                          exact={c.sidoName === '전국'}
                          to={ c.sidoName === '전국' ? '/api' : `/api/${c.sidoName}`}

                >{c.sidoName}</Category>
            ))}
        </CategoriesBlock>
    );
};

export default Categories;
```

categories라는 배열 안에 sidoName 값이 들어있는 객체를 카테고리로 사용합니다. 

react-router-dom의 `NavLink`를 사용하여 선택된 카테고리에 다른 스타일을 주었습니다. div, a, button, input처럼 일반 HTML요소가 아닌 특정 컴포넌트에 styled-components를 사용할 때는 styled(컴포넌트이름) 과 같은 형식을 사용하면 됩니다. 

NavLink로 만들어진 Category 컴포넌트에 to 값은 "/api/카테고리이름" 으로 설정해줍니다. 그리고 전국은 예외적으로 "/api" 로 설정했습니다. to 값이 "/api" 인 전국인 경우 exact값을 true로 설정해주어야 하는데, 이 값을 설정하지 않으면 다른 카테고리가 선택되었을 경우에도 전국 링크에 스타일이 적용되는 오류가 발생합니다. 



### AirPage 생성

카테고리를 추가하기 위해 components디렉터리에  AirPage.js 파일을 생성합니다. 

AirPage 컴포넌트는 Categories, AirList 컴포넌트를 함께 렌더링 해줍니다. 

```javascript
import React from 'react';
import Categories from './Categories';
import AirList from './AirList';

const AirPage = ({ match }) => {
    // 카테고리가 선택되지 않았으면 기본값 전국으로 사용
    const category = match.params.category || '전국';

    return (
        <>
            <Categories />
            <AirList category={category} />
        </>
    );
};

export default AirPage;
```

App.js에서 "/api" 요청시 AirList컴포넌트를 렌더링했지만, 카테고리와  AirList 컴포넌트가 포함된 AirPage컴포넌트를 렌더링하도록 수정할 것입니다. src/App.js 파일을 다음과 같이 수정합니다. 

```javascript
import React, {useState} from 'react';
import { Route , Link, Switch } from 'react-router-dom';
import About from './About';
import Home from './Home';
import Profiles from './Profiles';
import HistorySample from './HistorySample';
import axios from 'axios';
import AirPage from './components/AirPage';

const App = () => {


    const [data, setData] = useState(null);

    const onClick = async () => {
        try {
            const response = await axios.get(
                'http://3.35.219.122:3000/api?sidoName=경남',
            );
            setData(response.data);
        } catch (e) {
            console.log(e);
        }
    };


    return (
        <div>
            <ul>
                <li>
                    <Link to="/">홈</Link>
                </li>
                <li>
                <Link to="/about">소개</Link>
                </li>
                <li>
                    <Link to="/profiles">프로필</Link>
                </li>
                <li>
                    <Link to='/history'>History 예제</Link>
                </li>
                <li>
                    <Link to='/api'>Api 예제</Link>
                </li>
            </ul>
            <hr/>
            <Switch>
            <Route path="/" component={Home} exact={true} />
            <Route path={["/about", "/info"]} component={About} />
            <Route path="/profiles" component={Profiles} />
            <Route path="/history" component={HistorySample} />
            <Route path="/api/:category?" component={AirPage} />
                <Route
                    // path를 따로 정의하지 않으면 모든 상황에 렌더링됨
                    render={({ location }) => (
                        <div>
                            <h2>이 페이지는 존재하지 않습니다:</h2>
                            <p>{location.pathname}</p>
                        </div>
                    )}
                />
            </Switch>

            <div>
                <div>
                    <button onClick={onClick}>불러오기</button>
                </div>
                {data && <textarea rows={7} value={JSON.stringify(data, null, 2)} readOnly={true} />}
            </div>


        </div>
);
};

export default App;
```

위 코드에서 사용된 path에 /:category?와 같은 형태로 맨뒤에 물음표 문자가 들어가 있는데, 이것은 category값이 선택적이라는 것을 의미합니다. 즉 있을수도 없을수도 있다는 뜻입니다. 



API 호출시 카테고리를 지정해주기 위해 AirList.js 파일을 다음과 같이 수정합니다. AirPage컴포넌트로 부터 props로 받아온 category에 따라 카테고리를 지정하여 API를 요청하게 됩니다. 

```javascript
import React, {useState, useEffect} from 'react';
import styled from 'styled-components';
import AirItem from './AirItem';
import axios from 'axios';


const AirListBlock = styled.div`
  box-sizing: border-box;
  padding-bottom: 3rem;
  width: 768px;
  margin: 0 auto;
  margin-top: 2rem;
  @media screen and (max-width: 768px) {
    width: 100%;
    padding-left: 1rem;
    padding-right: 1rem;
  }
`;


const AirList = ({category}) => {

    const [articles, setArticles] = useState(null);
    const [loading, setLoading] = useState(false);

        useEffect(() => {
            // async를 사용하는 함수 따로 선언
            const fetchData = async () => {
                setLoading(true);
                try {
                    const response = await axios.get(
                    `http://localhost:3001/api?sidoName=${category}`,
                        );
                    setArticles(response.data.response.body.items);
                    //console.log(response.data.response.body.items)
                } catch (e) {
                    console.log(e);
                }
                setLoading(false);
            };
            fetchData();
        }, [category]);

        // 대기 중일 때
        if (loading) {
            return <AirListBlock>대기 중…</AirListBlock>;
        }
        // 아직 articles 값이 설정되지 않았을 때
        if (!articles) {
            return null;
        }
        // articles 값이 유효할 때
        return (
            <AirListBlock>
                {articles.map(article => (
                    <AirItem key={article.stationName} article={article} />
                ))}
            </AirListBlock>
        );
};



export default AirList;
```

현재 요청한 카테고리 값에 따라 요청 주소가 동적으로 변경되게 됩니다.  추가로 카테고리값이 변경될때마다 api를 새로 호출해야 하기 때문에 useEffect의 두번째 파라미터로 category를 넣어주어야 합니다. 이는 category가 변경될때 마다 useEffect가 실행되도록 해줍니다. 

여기까지 작업을 마쳤다면, 브라우저에서 http://localhost:3000/api으로 접속해서 카테고리 클릭 시 페이지 주소가 변경되고 대기오염 정보를 잘 출력하는지 확인하면 됩니다. 

![img](https://dbcore-assets-public.s3.amazonaws.com/tutorials/tutorial-nodejs-web-development/ch02/images/ch09-api-end.png)