<template lang="pug">
a.c-linear-schedule-session(:class="{faved}", :style="style", :href="link", @click="onSessionLinkClick($event, session)", :target="linkTarget")
	.time-box
		.start(:class="{'has-ampm': startTime.ampm}")
			.time {{ startTime.time }}
			.ampm(v-if="startTime.ampm") {{ startTime.ampm }}
		.duration {{ durationPretty }}
	.info
		.title {{ getLocalizedString(session.title) }}
		.speakers(v-if="session.speakers") {{ session.speakers.map(s => s.name).join(', ') }}
		.abstract(v-if="showAbstract", v-html="abstract")
		.bottom-info
			.track(v-if="session.track") {{ getLocalizedString(session.track.name) }}
			.room(v-if="showRoom && session.room") {{ getLocalizedString(session.room.name) }}
	bunt-icon-button.btn-fav-container(@click.prevent.stop="faved ? $emit('unfav', session.id) : $emit('fav', session.id)")
		svg.star(viewBox="0 0 24 24")
			path(d="M12,17.27L18.18,21L16.54,13.97L22,9.24L14.81,8.62L12,2L9.19,8.62L2,9.24L7.45,13.97L5.82,21L12,17.27Z")

</template>
<script>
import moment from 'moment-timezone'
import MarkdownIt from 'markdown-it'
import { getLocalizedString } from 'utils'

const markdownIt = MarkdownIt({
	linkify: true,
	breaks: true
})

export default {
	props: {
		session: Object,
		showAbstract: {
			type: Boolean,
			default: true
		},
		showRoom: {
			type: Boolean,
			default: true
		},
		faved: {
			type: Boolean,
			default: false
		}
	},
	inject: {
		eventUrl: { default: null },
		linkTarget: { default: '_self' },
		generateSessionLinkUrl: {
			default () {
				return ({eventUrl, session}) => `${eventUrl}talk/${session.id}/`
			}
		},
		onSessionLinkClick: {
			default () {
				return () => {}
			}
		}
	},
	data () {
		return {
			getLocalizedString
		}
	},
	computed: {
		link () {
			return this.generateSessionLinkUrl({eventUrl: this.eventUrl, session: this.session})
		},
		style () {
			return {
				'--track-color': this.session.track?.color || 'var(--pretalx-clr-primary)'
			}
		},
		startTime () {
			// check if 12h or 24h locale
			if (moment.localeData().longDateFormat('LT').endsWith(' A')) {
				return {
					time: this.session.start.format('h:mm'),
					ampm: this.session.start.format('A')
				}
			} else {
				return {
					time: moment(this.session.start).format('LT')
				}
			}
		},
		durationMinutes () {
			return moment(this.session.end).diff(this.session.start, 'minutes')
		},
		durationPretty () {
			let minutes = this.durationMinutes
			const hours = Math.floor(minutes / 60)
			if (minutes <= 60) {
				return `${minutes}min`
			}
			minutes = minutes % 60
			if (minutes) {
				return `${hours}h${minutes}min`
			}
			return `${hours}h`
		},
		abstract () {
			try {
				return markdownIt.renderInline(this.session.abstract)
			} catch (error) {
				return this.session.abstract
			}
		}
	}
}
</script>
<style lang="stylus">
.c-linear-schedule-session
	display: flex
	min-width: 300px
	min-height: 96px
	margin: 8px
	overflow: hidden
	color: $clr-primary-text-light
	position: relative
	.time-box
		width: 69px
		box-sizing: border-box
		background-color: var(--track-color)
		padding: 12px 16px 8px 12px
		border-radius: 6px 0 0 6px
		display: flex
		flex-direction: column
		align-items: center
		.start
			color: $clr-primary-text-dark
			font-size: 16px
			font-weight: 600
			margin-bottom: 8px
			display: flex
			flex-direction: column
			align-items: flex-end
			&.has-ampm
				align-self: stretch
			.ampm
				font-weight: 400
				font-size: 13px
		.duration
			color: $clr-secondary-text-dark
	.info
		flex: auto
		display: flex
		flex-direction: column
		padding: 8px
		border: border-separator()
		border-left: none
		border-radius: 0 6px 6px 0
		background-color: $clr-white
		min-width: 0
		.title
			font-size: 16px
			font-weight: 500
			margin-bottom: 4px
		.speakers
			color: $clr-secondary-text-light
		.abstract
			margin: 8px 0 12px 0
			// TODO make this take up more space if available?
			display: -webkit-box
			-webkit-line-clamp: 3
			-webkit-box-orient: vertical
			overflow: hidden
		.bottom-info
			flex: auto
			display: flex
			align-items: flex-end
			.track
				flex: 1
				color: var(--track-color)
				ellipsis()
				margin-right: 4px
			.room
				flex: 1
				text-align: right
				color: $clr-secondary-text-light
				ellipsis()
	.btn-fav-container
		display: none
		position: absolute
		top: 2px
		right: 2px
		icon-button-style(style: clear)
		svg path
			fill: none
			stroke: $clr-primary-text-light
			stroke-width: 1px
			vector-effect: non-scaling-stroke
	&.faved
		.btn-fav-container
			display: inline-flex
			svg path
				fill: $clr-primary-text-light
	&:hover
		.info
			border: 1px solid var(--track-color)
			border-left: none
			.title
				color: var(--pretalx-clr-primary)
		.btn-fav-container
			display: inline-flex
	// +below('m')
	// 	min-width: 0
</style>
