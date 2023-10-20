In the vast landscape of digital protocols, there exists a hidden gem often overshadowed by its more mainstream counterparts: the CalDAV protocol. While lesser known, I firmly believe that CalDAV holds immense potential, particularly when harnessed for automation purposes. In this article, I will share my perspective on CalDAV's untapped capabilities and walk you through innovative ways to seamlessly incorporate it into your automation workflows, transforming an underrated protocol into a powerful tool for optimizing your daily tasks.

Nice introduction ChatGPT! I didn't know how to start this post, so I went to ChatGPT and asked him for introduction. I will probably never write any sentence even remotely as good as this, so I just took it and paste it. But there is an actual reason why I make this post.

I don't even remember how I learned about CalDAV protocol, but it was always a mystery to me. Recently I started to playing with it, and it surprised me how easy it is but also how complex it is at the same time.

So how is that mystery protocol easy? HTTP! Even though HTTP is a separate protocol and has nothing to do with CalDAV (at least to my knowledge) it uses similar concepts at its core. It is actually so similar that we can use POSTMAN to do CalDAV requests...

<img src="https://cms.filipizydorczyk.pl/api/v1/media/Pasted image 20230821175501.png">

This example shows how to call CalDAV endpoint exposed by Nextcloud. You need to additionally add headers for Basic Auth to get authorization, and you also need to add header `Depth: 1`. Couple of things to unpack here:

-   `REPORT` method - as I said it's not really HTTP protocol and in the default list of methods in POSTMAN you will not find this method. It's an CalDAV endpoint, and you will need to type it yourself with a keyboard.
-   Body is an xml - and this is what I meant, saying it's easy and hard at the same time. Making a request was easy, and it's not really different from calling the API from your application, but to get the result or filter them you need to provide an XML body that match the specification. It's not that hard, but it's not intuitive either and to get this basic XML I just asked Chat GTP to give me one. The specification is pretty long and there are not many tutorials on a subject, but even if you don't want to become CalDAV expert, you should be able to figure it out.
-   Headers - you will need to add some headers to the request for: basic auth, `Depth: 1` for content in the response and `Content-Type: application/xml` for body
    As you may see it is very similar to HTTP protocol and because of that I like thinking about it as a customized HTTP. Now when we know we can basically use HTTP tools to call CalDAV protocol we can push it even further and call it programmatically using library like **Axios**

```ts
import axios from "axios";

const resp = await axios({
    method: "REPORT",
    url,
    auth: { username, password },
    headers: { Depth: "1", "Content-Type": "application/xml" },
    data: /* XML */ `<?xml version="1.0" encoding="utf-8"?>
        <C:calendar-query xmlns:D="DAV:" xmlns:C="urn:ietf:params:xml:ns:caldav">
            <D:prop>
                <D:getetag />
                <C:calendar-data />
            </D:prop>
            <C:filter>
                <C:comp-filter name="VCALENDAR">
                    <C:comp-filter name="VEVENT">
                        <C:time-range start="20230814T000000Z" />
                    </C:comp-filter>
                </C:comp-filter>
            </C:filter>
        </C:calendar-query>`,
});
```

So that's what I did. I fetched my calendar events with this endpoint and I decided to create POC with this.

**Offtop**: Notice that I am using string to provide body. It's fine, especially with a prettier and `/* XML */` comment that will format this string to be readable, but a better option would be to import an XML file to the string and use that. But that's something I want to address with the next post. The response is basically XML with some string that are `ics` formatted events.

```xml
<?xml version="1.0"?>
<d:multistatus xmlns:d="DAV:" xmlns:s="http://sabredav.org/ns" xmlns:cal="urn:ietf:params:xml:ns:caldav" xmlns:cs="http://calendarserver.org/ns/" xmlns:oc="http://owncloud.org/ns" xmlns:nc="http://nextcloud.org/ns">
    <d:response>
        <d:href>/remote.php/dav/calendars/username/calendar/id.ics</d:href>
        <d:propstat>
            <d:prop>
                <d:getetag>&quot;tag&quot;</d:getetag>
                <cal:calendar-data>BEGIN:VCALENDAR
CALSCALE:GREGORIAN
VERSION:2.0
PRODID:-//SabreDAV//SabreDAV//EN
BEGIN:VEVENT
CREATED:20220603T114724Z
DTSTAMP:20220603T114807Z

...

```

So to actually use it we are going to need two more things:

-   xml mapper
-   ics mapper
    First, we are going to map response XML to JSON object. To do so, I used `xml2js` library.

```ts
const XmlMapper = {
    parseXmlStringToJson: async (content: string): Promise<Object> => {
        return new Promise((resolve, reject) => {
            parseString(content, (err, result) => {
                if (err) {
                    reject(err);
                }

                resolve(result);
            });
        });
    },
};
```

Pretty easy right? The next part is much more "dirty" because we need to navigate this XML structure to get `ics`'es and map them as well. This is the code I got

```ts
parseCalDavString: async (data: string): Promise<CalednarEntity[]> => {
        const parsedXml: any = await XmlMapper.parseXmlStringToJson(data);
        const response: CalednarEntity[] = [];
        let id = 0;

        if (
            !parsedXml?.["d:multistatus"]?.["d:response"] ||
            !parsedXml?.["d:multistatus"]?.["d:response"].length
        ) {
            Promise.reject(
                "Parsed xml is not present or is in incorrect format!"
            );
        }

        parsedXml["d:multistatus"]["d:response"].forEach((element: any) => {
            const entity = ical.sync.parseICS(
                element["d:propstat"][0]["d:prop"][0]["cal:calendar-data"][0]
            );
            const uuid = Object.keys(entity)[0];
            const calendarData: any = entity[uuid];

            response.push({
                id,
                uuid,
                title: calendarData.summary,
                start_time: moment(new Date(calendarData.start)),
                end_time: moment(new Date(calendarData.end)),
            });

            id = id + 1;
        });

        return response;
    },
```

Not that nice anymore... We can provide some types with typescript, but I am not sure if the response will always look like that (I mean for every type of request) and it is just PoC, so I left it this way. What we should do in real application is to check for error code and if the request was successful run some type check function to validate body response. If it passes, we can use the data, if it doesn't, we should throw an exception.

But that's pretty much it now I can pass this data to react component to see my events.

```tsx
const App = () => {
    const [items, setItems] = useState<any[]>([]);

    useEffect(() => {
        CalDavService.getAllEvents(
            process.env.REACT_APP_NEXTCLOUD_CALENDAR_URL || ""
        ).then((response) => {
            setItems(response.map((item) => ({ ...item, group: 1 })));
        });
    }, []);

    return (
        <ReactTimeline
            groups={[{ id: 1, title: "Wydarzenia" }]}
            items={items}
            defaultTimeStart={moment().add(-12, "hour")}
            defaultTimeEnd={moment().add(12, "hour")}
        />
    );
};
```

Obviously, this very simple example that will not benefit you in any sort of way. But using more complex filter and keeping some conventions in your calendars could allow you to create some custom functionality that your calendar app is missing without writing the whole calendar app from scratch. When you have the app ready, you can even compile it with your credentials hard-coded with will simplify your user experience. You are not going to share this app with anyone anyway... In this example, I used React with Electron (which required me to do some gymnastic with configuration to make all libraries work, and you are forced to use electron to solve the CORS issue). You can see this PoC in my repositories, maybe some of it will be useful.

Doing web app is good idea because it would allow you to go cross-platform very easy, but it required me to do some configurations, but you can use more native technologies that may you save some of this time if you are decided what platform you want to go with or if you have server that you might host on you can even create REST APP that will act as an abstraction between your calendar and your app. However, this is another pile of work because then you would need some authorization not to expose your calendars to anyone.

Sources: [NPM: React calendar](https://www.npmjs.com/package/react-calendar-timeline#react-calendar-timeline), [stackoverflow:Module not found: Error: Can't resolve 'timers'](https://stackoverflow.com/questions/66859193/module-not-found-error-cant-resolve-timers), [bobbyhadz: Module not found: Can't resolve 'fs' error](https://bobbyhadz.com/blog/module-not-found-cant-resolve-fs), [stackoverflow: where is create-react-app webpack config and files?](https://stackoverflow.com/questions/48395804/where-is-create-react-app-webpack-config-and-files). [NPM: @craco/craco](https://www.npmjs.com/package/@craco/craco), [github:CORS Problem CalDav](https://github.com/nextcloud/server/issues/19719), [stackoverflow: can I make the browser ignore the (CORS) rules?](https://stackoverflow.com/questions/36967788/can-i-make-the-browser-ignore-the-cors-rules), [stackoverflow: How do you handle CORS in an electron app?](https://stackoverflow.com/questions/51254618/how-do-you-handle-cors-in-an-electron-app), [pratikpc.medium: Bypassing CORS with Electron](https://pratikpc.medium.com/bypassing-cors-with-electron-ab7eaf331605), [Getting Started with Electron by Creating a React App](https://www.section.io/engineering-education/desktop-application-with-react/), [github: RpcIpcMessagePortClosedError: Cannot send the message - the message port has been closed for the process 145068](https://github.com/nrwl/nx/issues/8150), [JavaScript Date and moment.js](https://golb.hplar.ch/2017/01/JavaScript-Date-and-moment-js.html)[CalDAV specification](https://datatracker.ietf.org/doc/html/rfc4791)
