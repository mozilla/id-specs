## Relying Party (RP) API

<dl>
    <dt><h3>navigator.id.watch( <em>parameters</em> )</h3></dt>
    <dd>
        <p>
            Configure BrowserID by registering callbacks and setting display options.
        </p>
        <blockquote>
            <strong>Implementation Notes:</strong>
            <ul>
                <li>May only be called once. Subsequent calls must be ignored and should log an error.</li>
                <li>User input should be treated carefully &mdash; siteLogo may be an SVG, which opens an avenue for scripting.</li>
            </ul>
        </blockquote>

        <strong>Required Parameters:</strong>
        <dl>
            <dt>
                <strong><code>onlogin</code></strong>:
                <code><strong>function (assertion)</strong> { … }</code>
            </dt>
            <dd>
                Callback.
                Invoked when a user attempts to sign in.
                Receives a Backed Identity Assertion as its first argument.
            </dd>
        </dl>

        <strong>Optional Parameters:</strong>
        <dl>
            <dt>
                <strong><code>onready</code></strong>:
                <code><strong>function ()</strong> { … }</code>
            </dt>
            <dd>
                Callback.
                Invoked when the user agent is ready to handle BrowserID API calls.
            </dd>
        </dl>

        <strong>Visual Customization (Optional):</strong>
        <dl>
            <dt><strong><code>siteName</code></strong>: <strong>String</strong> (Freeform text)</dt>
            <dd>Human-friendly name of the Relying Party.</dd>

            <dt><strong><code>siteLogo</code></strong>: <strong>String</strong> (URL, absolute path, or <code>data:</code> URI)</dt>
            <dd>Image that represents the Relying Party.</dd>

            <dt><strong><code>backgroundColor</code></strong>: <strong>String</strong> (Hex #rgb or #rrggbb)</dt>
            <dd>Background color for displaying Relying Party information.</dd>
        </dl>
    </dd>

    <dt><h3>navigator.id.request( <em>parameters</em> )</h3></dt>
    <dd>
        <p>
            Prompt the user to select an email address and sign into the Relying Party.
            Upon successful completion, a Backed Identity Assertion is passed to the <code>onlogin</code> callback registered in <code>navigator.id.watch()</code>.
        </p>
        <blockquote>
            <strong>Implementation Notes:</strong>
            <ul>
                <li>Must be called in response to direct user action, such as a click.</li>
                <li>Raises an error if called before <code>navigator.id.watch()</code> has been invoked.</li>
            </ul>
        </blockquote>
        <strong>Behavior Customization (Optional):</strong>
        <dl>
            <dt><strong><code>email</code></strong>: <strong>String</strong> (Email address)</dt>
            <dd>Skip address selection and attempt to authenticate the user as the given email address.</dd>

            <dt><strong><code>oncancel</code></strong>: <code><strong>function ()</strong> { … }</code></dt>
            <dd>
                Callback.
                Invoked if the user closes the login prompt without selecting an address.
            </dd>
        </dl>
    </dd>
</dl>