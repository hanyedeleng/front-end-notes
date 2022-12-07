### jest mock:

https://reactjs.org/docs/testing-recipes.html

#### Mock data and mock modules:

https://www.valentinog.com/blog/testing-react/
```
describe("User component", () => {
  test("it shows a list of users", async () => {
    const fakeResponse = [{ name: "John Doe" }, { name: "Kevin Mitnick" }];
    jest.spyOn(window, "fetch").mockImplementation(() => {
      const fetchResponse = {
        json: () => Promise.resolve(fakeResponse)
      };
      return Promise.resolve(fetchResponse);
    });
    await act(async () => {
      render(<Users />, container);
    });
    
    expect(container.textContent).toBe("John DoeKevin Mitnick");
    window.fetch.mockRestore();
  });
});
```


#### how to mock a default function from another file"
```
import * as AddRoute from "Foundation/Services/PasSelfScheduling/addTokenToRoute";
const mock = jest.spyOn(AddRoute, "default");
mock.mockImplementation(() => Promise.resolve("jwt"));

or ----
import * as helper from "../../../../common/helper";
const mock = jest.spyOn(helper, "ReplaceSelfSchedulingLink");
mock.mockImplementation(() => Promise.resolve("jwt"));

```


#### The mocked file looks like this:
```
const addTokenToRoute = async (route, TextBeingReplaced, ReplacedText) => {
  let routeWithJWT;
  await PasSchedulingClient.getjwtToken().then((data) => {
    routeWithJWT = route.replace(
      TextBeingReplaced,
      ReplacedText + data.access_token
    );
  });
  return routeWithJWT;
};

export const ReplaceSelfSchedulingLink = async (html, surveyResultData) => {
  const htmlStr = html;

  const tmpDiv = document.createElement("div");
  tmpDiv.innerHTML = htmlStr;

  const anchor = tmpDiv.getElementsByTagName("a")[0];

  let route;
  if (!anchor) {
    return;
  } else {
    route = anchor.getAttribute("href");
  }

  const jwtRouteMatch = route.match(/jwt=([^&]*)/);
  const ttqRouteMatch = route.match(/ttqoutcome=([^&]*)/);

  if (jwtRouteMatch && jwtRouteMatch[1] === "true" && !ttqRouteMatch) {
    const routeWithToken = await addTokenToRoute(route, "jwt=true", "jwt=");
    const hrefMatch = htmlStr.match(/href="(.*?)"/);
    const updatedhtml = htmlStr.replace(hrefMatch[1], routeWithToken);

    return updatedhtml;
  } else if (
    jwtRouteMatch &&
    jwtRouteMatch[1] === "true" &&
    ttqRouteMatch &&
    ttqRouteMatch[1] === "true"
  ) {
    const routeWithToken = await addTokenToRoute(route, "jwt=true", "jwt=");

    let parsedResultObject;
    if (surveyResultData.result.description) {
      parsedResultObject = JSON.parse(
        surveyResultData.result.description.replace(/\s+/g, " ").trim()
      );
    }

    if (!parsedResultObject.Genes) {
      delete parsedResultObject.Genes;
    }
    if (!parsedResultObject.DiseasePanel) {
      delete parsedResultObject.DiseasePanel;
    }

    const result = surveyResultData.result.result;
    let stringResultObject = Object.entries(parsedResultObject)
      .map(([key, val]) => `${key}=${val}`)
      .join(";");

    stringResultObject = stringResultObject
      ? `${result};${stringResultObject}`
      : `${result}`;

    const encodedURI = encodeURI(stringResultObject.trim());

    if (routeWithToken) {
      const updatedRoute = routeWithToken.replace(
        "ttqoutcome=true",
        `ttqoutcome=${encodedURI}`
      );

      const hrefMatch = htmlStr.match(/href="(.*?)"/);

      const updatedhtml = htmlStr.replace(hrefMatch[1], updatedRoute);

      return updatedhtml;
    }
  }
};

```



#### how to mock a class component -- api

```
import surveyService from "../../../../Foundation/Services/Engage/SurveyService";

jest.mock("../../../../Foundation/Services/Engage/SurveyService");
// eslint-disable-next-line react/display-name
jest.mock("@engage/engage-app", () => () => <div>Engage App</div>);

const spy = jest.spyOn(surveyService, "getSurveyStatus");
spy.mockImplementation(() => {
	return new Promise((resolve) => {
		resolve({
			data: { result: { status: "NotAssigned", id: "1177" } },
			status: 200,
			statusText: "OK",
			headers: "",
			config: {},
		});
	});
});
```




Difference: don't need to jest.mock(module) for functions, but do need it for class.

