<script>
	import { brand } from '$lib/config';
</script>

<nav class="container-fluid">
	<ul>
		<li>
			<a href="/" class="contrast"><strong>{brand.name}</strong></a>
		</li>
	</ul>
	<ul>
		<li>
			<details role="list" dir="rtl">
				<summary aria-haspopup="listbox" role="link" class="secondary">Theme</summary>
				<ul role="listbox">
					<li><a href="#" data-theme-switcher="auto">Auto</a></li>
					<li><a href="#" data-theme-switcher="light">Light</a></li>
					<li><a href="#" data-theme-switcher="dark">Dark</a></li>
				</ul>
			</details>
		</li>
		<li>
			<details role="list" dir="rtl">
				<summary aria-haspopup="listbox" role="link" class="secondary">Examples (v1)</summary>
				<ul role="listbox">
					<li><a href="../v1-preview/">Preview</a></li>
					<li><a href="../v1-preview-rtl/">Right-to-left</a></li>
					<li><a href="../v1-classless/">Classless</a></li>
					<li><a href="../v1-basic-template/">Basic template</a></li>
					<li><a href="../v1-company/">Company</a></li>
					<li><a href="../v1-google-amp/">Google Amp</a></li>
					<li><a href="../v1-sign-in/">Sign in</a></li>
					<li><a href="../v1-bootstrap-grid/">Bootstrap grid</a></li>
				</ul>
			</details>
		</li>
	</ul>
</nav>

<slot />
