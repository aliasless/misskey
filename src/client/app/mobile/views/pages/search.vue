<template>
<mk-ui>
	<template #header><fa icon="search"/> {{ q }}</template>

	<main>
		<mk-notes ref="timeline" :make-promise="makePromise" @inited="inited"/>
	</main>
</mk-ui>
</template>

<script lang="ts">
import Vue from 'vue';
import i18n from '../../../i18n';
import Progress from '../../../common/scripts/loading';
import { genSearchQuery } from '../../../common/scripts/gen-search-query';

const limit = 20;

export default Vue.extend({
	i18n: i18n('mobile/views/pages/search.vue'),
	data() {
		return {
			makePromise: async cursor => this.$root.api('notes/search', {
				limit: limit + 1,
				untilId: cursor ? cursor : undefined,
				...(await genSearchQuery(this, this.q))
			}).then(notes => {
				if (notes.length == limit + 1) {
					notes.pop();
					return {
						notes: notes,
						more: true
					};
				} else {
					return {
						notes: notes,
						more: false
					};
				}
			})
		};
	},
	watch: {
		$route() {
			this.$refs.timeline.reload();
		}
	},
	computed: {
		q(): string {
			return this.$route.query.q;
		}
	},
	mounted() {
		document.title = `${this.$t('search')}: ${this.q} | ${this.$root.instanceName}`;
	},
	methods: {
		inited() {
			Progress.done();
		},
	}
});
</script>
