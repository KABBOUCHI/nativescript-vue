<template>
	<stack-layout>
		<list-view :items="items" class="list-group" :templateSelector="templateSelector" separatorColor="red" @itemTap="onItemTap" @loaded="onLoaded" @loadMoreItems="onLoadMoreItems">
			<template scope="item">
				<stack-layout orientation="horizontal" class="list-group-item">
					<image :src="item.image" class="thumb"></image>
					<stack-layout>
						<label class="list-group-item-heading" :text="item.title" textWrap="true"></label>
						<label class="list-group-item-text" text="The rest of the content" textWrap="true"></label>
					</stack-layout>
				</stack-layout>
			</template>
			<template name="page" scope="item">
				<stack-layout orientation="horizontal" class="list-group-item active">
					<label :text="item.title"></label>
				</stack-layout>
			</template>
		</list-view>
	</stack-layout>
</template>
<script type="text/javascript">

	const Page = require('ui/page').Page
	const StackLayout = require('ui/layouts/stack-layout').StackLayout
	const ScrollView = require('ui/scroll-view').ScrollView
	const Image = require('ui/image').Image
	const Label = require('ui/label').Label
	const Button = require('ui/button').Button

	export default{
		data (){

			return{
				subreddit: '/r/nativescript',
				page_num: 1,
				last_page: '',
				items: []
			}
		},
		created() {
			this.fetchItems()
		},

		methods: {
			onItemTap(e) {
				let item = this.items[e.index]
				let detailsPage = new Page()
				let layout = new StackLayout()
				let scroller = new ScrollView()
				scroller.content = layout

				let label = new Label()
				label.text = item.title
				label.className = 'h2'
				label.textAlignment = 'center'

				let image = new Image()
				image.src = item.fullImage

				let closeButton = new Button()
				closeButton.text = 'Close'
				closeButton.on('tap', () => detailsPage.closeModal())

				layout.addChild(label)
				layout.addChild(image)
				layout.addChild(closeButton)

				detailsPage.content = scroller
				this.$parent.$refs.page.nativeView.showModal(detailsPage)
			},

			onLoaded(e) {
				console.log('The list has been loaded')
			},

			onLoadMoreItems(e) {
				console.log('Loading more items')
				return this.fetchItems()
			},

			templateSelector(item) {
				return item.type === 'page' ? 'page' : 'default'
			},

			navigationButtonPressed() {
				console.log(">>> navigation button pressed (but nothing to do really..)")
			},

			refresh() {
				this.items = []
				this.page_num = 1
				this.fetchItems()
			},

			chooseSubreddit() {
				prompt({
					title: 'Change subreddit:',
					defaultText: this.subreddit,
					okButtonText: 'Ok',
					cancelButtonText: 'Cancel',
				}).then((r) => {
					if (r.result) {
						this.subreddit = r.text
						this.refresh()
					}
				})
			},

			fetchItems() {

				var url = `https://www.reddit.com/${this.subreddit}.json?limit=10&count=10&after=${this.last_page}`;


				fetch(url).then(response => { return response.json(); }).then( (res) => {
    				
    				this.items.push({
						title: 'Page ' + this.page_num,
						type: 'page'
					})
					res.data.children.forEach((item) => {
						this.items.push({
							title: item.data.title,
							image: item.data.thumbnail,
							fullImage: item.data.preview.images[0].source.url,
							type: 'entry'
						})
					})
					this.last_page = res.data.after
					this.page_num++;

					console.log('Loaded more items')
				}).catch((err) => {
					console.log('err..' + err)
				})
			}
		}
	}
</script>
