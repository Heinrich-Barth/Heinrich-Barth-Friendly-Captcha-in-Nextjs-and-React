# Using FriendlyCaptcha in Next.js/React

Although the theory is rather simple, there are many examples on the internet that do not quite show how to integrate FriendlyCaptcha correctly, because they result in a rendering error in the frontend or in a warning on the backend.

Therefore, this little `readme` will cover it.

## Prerequisites

Before you start:

* Creat your FriendlyCaptcha account to obtain a key
* Install the FriendlyCaptcha sdk 

## Building the page component

The page component is a skeleton and simply loads our react captcha dynamically. Once the solution has been received, a callback function will receive it for further use in the form. This is omitted here, of course.

```JSX
"use client"

import dynamic from "next/dynamic";

/** Simply load the captcha dynamically to avoid rendering errors and build failures */
const FriendlyCaptcha = dynamic(() => import("./FriendlyCaptcha"), { ssr: false })

export default function PageComponentWithCaptcha()
{
    return <div>
        <form>
            <FriendlyCaptcha 
                onSuccess={(solution:string) => console.info("Work with solution in your react app:", solution)} 
                onError={(message:string) => console.warn(message)}
            />
        </form>
    </div>
}
```

## Building the actual FriendlyCaptcha component

This example is based on the following resources:
* https://docs.friendlycaptcha.com/#/friendlyCaptchaWidget_api?id=full-example-in-react-with-react-hooks 
* https://developer.friendlycaptcha.com/docs/v2/getting-started/install#create-friendlyCaptchaWidgets-from-your-code
* https://developer.friendlycaptcha.com/docs/v2/guides/upgrading-from-v1/javascript-api

```JSX
import React from "react";
import { FriendlyCaptchaSDK } from "@friendlycaptcha/sdk";

const CAPTCHA_KEY = "LOAD THIS ANY WAY YOU WANT";

export default function FriendlyCaptcha({onSuccess, onError}:{ onSuccess:Function, onError:Function}) 
{
    const targetContainer = React.useRef(null);
    const friendlyCaptchaWidget = React.useRef<any>(null);

    const setupCaptchaInstance = function()
    {
        const sdk = new FriendlyCaptchaSDK();

        friendlyCaptchaWidget.current = sdk.createfriendlyCaptchaWidget({
            element: targetContainer.current,
            sitekey: CAPTCHA_KEY,
            startMode: "focus",
        });

        friendlyCaptchaWidget.current.addEventListener("frc:friendlyCaptchaWidget.complete", (event:any) => onSuccess(event.detail?.response ?? ""));        
        friendlyCaptchaWidget.current.addEventListener("frc:friendlyCaptchaWidget.error", (event:any) => onError(event.detail.error));
        friendlyCaptchaWidget.current.addEventListener("frc:friendlyCaptchaWidget.expire", () => onError("Captcha solution expired"));
    }

    React.useEffect(() => {

        if (!friendlyCaptchaWidget.current  && targetContainer.current) 
            setupCaptchaInstance();
  
        return () => {
            if (friendlyCaptchaWidget.current) 
                (friendlyCaptchaWidget.current as any).reset();
        };

    }, [targetContainer]);
    
    return <div ref={targetContainer} className="frc-captcha" />;
};
```
