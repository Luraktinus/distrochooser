<template>
	<div class="column col-3 hide-xs">
		<div class="column col-11">
			<ul class="menu left-box hide-xs fixed">
				<li class="menu-item">
					<a :href="'/info/'+this.globals.lang+'/about'">
						<i class="icon icon-link"></i> {{ text("sys.about") }}
					</a>
				</li>
				<li class="menu-item">
					<a :href="'/info/'+this.globals.lang+'/privacy'">
						<i class="icon icon-link"></i> {{ text("sys.privacy") }}
					</a>
				</li>
				<li class="menu-item">
					<a :href="'/info/'+this.globals.lang+'/contact'">
						<i class="icon icon-link"></i> {{ text("sys.contact") }}
					</a>
				</li>

				<li class="menu-item">
					<a target="_blank" href="https://blog.distrochooser.de">
						<i class="icon icon-link"></i> Blog
					</a>
				</li>
				<li class="menu-item">
					<a target="_blank" href="http://0fury.de">
					<img class="vendor" alt="0fury.de" src="/0fury.ico">
					{{ text("sys.vendor") }} <span class="0fury">0fury.de</span>
					</a>
				</li>
				<!-- menu item with badge -->
				<li class="menu-item">
					<div class="menu-badge">
						<a class="flag" title="Zur deutschen Version wechseln" href="/">
							<img src="/de.png" alt="Deutsch">
						</a>
						<a class="flag" title="Switch to english version" href="/en/">
							<img src="/gb.png" alt="English">
						</a>
						<a class="flag" title="Switch to chinese (simplified) version" href="/zh-cn/">
                                                        <img src="/zh-cn.png" alt="Chinese (simplified)">
                                                </a>

					</div>
					<a href="#">
						<i class="icon"></i> 
					</a>
				</li>
			</ul>
		</div>
		<div class="col-1">
		</div>
	</div>
</template>
 
<script>
import i18n from '../mixins/i18n'
import nuxt from '../mixins/nuxt-wrapper'
export default {
  mixins: [i18n, nuxt]
}
</script>
 
<style scoped>
.button-left-nav {
    color: #fff;
    display: block;
    width: 70%;    
		padding: 9px 12px 7px;
    font-size: 12px;
    font-weight: 700;
    text-transform: uppercase;
    margin-bottom: 0.5em;
}
.vendor{
    float: left;
    margin-right: 0.2em;
    margin-left: -0.1em;
}
.flag{
    margin-right: 0.5em;
}
.flag img{
    height: 1.3em;
}
.left-box{
    width: 20%;
}
</style>
