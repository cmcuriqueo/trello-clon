<template>
	<v-container>
		<draggable id="board" class="row pa-4 pt-10" v-model="columns" group="columns" ghost-class="ghost"
			draggable=".isDraggable" :delay="200" :delay-on-touch-only="true">
			<div class="cardColumm mr-3 isDraggable widthColumn" v-for="(column, indexColumn) in columns"
				:key="indexColumn">
				<v-card-text class="d-flex mb-0 mb-0 hoverGrab">
					<v-btn text>
						{{ column.name }}
					</v-btn>
					<v-spacer></v-spacer>
					<div class="text-center">
						<v-menu transition="scale-transition" origin="center center">
							<template v-slot:activator="{ on, attrs }">
								<v-btn icon v-bind="attrs" v-on="on">
									<v-icon>mdi-dots-horizontal</v-icon>
								</v-btn>
							</template>
							<v-list style="min-width: 300px;">
								<v-list-item>
									<v-list-item-content>
										<v-list-item-title class="text-center">Lista de Acciones</v-list-item-title>
									</v-list-item-content>

									<v-list-item-action>
										<v-btn :class="fav ? 'red--text' : ''" icon @click="fav = !fav">
											<v-icon>mdi-close</v-icon>
										</v-btn>
									</v-list-item-action>
								</v-list-item>
								<v-list-item @click="() => { }">
									<v-list-item-title v-text="'Agregar Card'"></v-list-item-title>
								</v-list-item>
								<v-list-item @click="() => { }">
									<v-list-item-title v-text="'Ordenar Por'"></v-list-item-title>
								</v-list-item>
								<v-list-item @click="() => { }">
									<v-list-item-title v-text="'Archivar esta Lista'"></v-list-item-title>
								</v-list-item>
							</v-list>
						</v-menu>
					</div>
				</v-card-text>
				<draggable v-model="column.cards" group="cards" @start="drag = true" @end="drag = false" :list="column.cards"
					ghost-class="ghost" class="v-card__text minHeight scrollBar cardColummContent"
					:id="'idColumn' + column.id" :delay="200" :delay-on-touch-only="true">
					<v-flex v-for="(card) in column.cards" :key="card.id">
						<v-card shaped class="mb-2 hoverGrab" @mouseover="card.hover = true"
							@mouseleave="card.hover = false" elevation="1" v-if="!card.editable" tile>
							<v-card-title>
								<v-btn text class="pa-0">
									{{ card.name }}

								</v-btn>
								<v-spacer></v-spacer>

								<v-btn icon v-if="card.hover" @click="onShowTask(card)">
									<v-icon>mdi-pencil</v-icon>
								</v-btn>

							</v-card-title>
						</v-card>
						<v-layout v-else justify-space-between class="mb-2">
							<v-text-field placeholder="Introduzca el título" solo v-model="card.name"
								v-on:keyup.enter="onAddConfirmCard(column)" :ref="'inputAddCard' + column.id"
								v-on:blur="onAddConfirmCard(column)"></v-text-field>
						</v-layout>
					</v-flex>

				</draggable>

				<v-card-actions class="pa-3" v-if="!column.isAddingCard">
					<v-btn block @click="addCard(column)" class="button-add-card justify-start text-center" depressed
						outlined>
						<v-icon>mdi-plus</v-icon> Nuevo Card
					</v-btn>
				</v-card-actions>


				<v-card-actions class="pa-3" v-else>
					<v-btn color="success" @click="onAddConfirmCard(column)" depressed>
						<v-icon>mdi-plus</v-icon> Agregar
					</v-btn>
					<v-btn text @click="onAddConfirmCard(column)" depressed>
						<v-icon>mdi-close</v-icon>
					</v-btn>
				</v-card-actions>
			</div>


			<v-btn @click="addColumn" v-if="!isAddingColumn" class="cardColumm widthColumn">
				<v-icon>mdi-add</v-icon> Agregar Columna
			</v-btn>

			<span class="pa-0 widthColumn">
				<v-text-field v-show="isAddingColumn" placeholder="Introduzca el título" solo v-model="nameColumn"
					append-icon="mdi-plus" @click:append="confirmAddColumn" class="pa-0 ma-0"
					v-on:keyup.enter="confirmAddColumn" ref="inputAddColumn"></v-text-field>
			</span>
		</draggable>





		<v-dialog v-model="dialogShowTask" persistent max-width="1000px">
			<v-card>
				<div v-if="Object.keys(selectedTaskShow).length > 0">
					<v-card-title>
						<span class="headline">{{ selectedTaskShow.name ? selectedTaskShow.name : "Sin título" }}</span>
					</v-card-title>
					<v-card-text class="pb-0">
						<p class="font-italic">Creado el 30/10/2019 16:07</p>
					</v-card-text>
					<v-card-text class="pt-0">
						<v-container fluid class="pa-0">
							<v-row>
								<v-col cols="12" md="8">
									<v-row>
										<v-col cols="12" md="12" v-if="testCheckList">
											<v-list>
												<v-subheader>Checklist</v-subheader>
												<v-progress-linear value="15"></v-progress-linear>
												<v-list-item-group multiple active-class="">
													<v-list-item>
														<template v-slot:default="{ active }">
															<v-list-item-action>
																<v-checkbox v-model="active"></v-checkbox>
															</v-list-item-action>

															<v-list-item-content>
																<v-list-item-title>Notificaciones</v-list-item-title>
															</v-list-item-content>
														</template>
													</v-list-item>
												</v-list-item-group>
											</v-list>
										</v-col>
										<v-col cols="12" md="12">
											<div class="text-h6">Descripción</div>
										</v-col>
										<v-col cols="12" md="12">
											<p v-html="selectedTaskShow.description" class="clickable"
												v-if="!editDescription" @click="editDescription = !editDescription; newDescription = selectedTaskShow.description"></p>
											<div v-if="editDescription">
												<ckeditor :editor="editor" v-model="newDescription"
													:config="editorConfig"></ckeditor>
											</div>

										</v-col>
										<v-col class="text-left pt-0" cols="12" md="12" v-if="editDescription">
											<v-btn color="primary" @click="selectedTaskShow.description = newDescription; editDescription = false;">
												Guardar
											</v-btn>
											<v-btn text @click="editDescription = false;">
												Cancelar
											</v-btn>
										</v-col>

										<v-col cols="12" md="12">
											<div class="text-h6">Actividad</div>
										</v-col>
										<v-col cols="12" md="12">
											<ckeditor :editor="editor" v-model="selectedTaskShow.actividad"
												:config="editorConfig"></ckeditor>
										</v-col>
										<v-col class="text-left pt-0" cols="12" md="12">
											<v-btn color="primary" @click="addActivity()">Guardar</v-btn>
										</v-col>
										<v-col cols="12" md="12">
											<div v-for="actividad in selectedTaskShow.listaActividad"
												v-bind:key="actividad.id">

												<p class="text-overline">User <span class="text-caption">{{
													formattedDate(actividad.time_create) }}</span></p>

												<v-alert color="blue-grey" dark dense prominent v-html="actividad.text">
												</v-alert>
											</div>

										</v-col>




									</v-row>
								</v-col>
								<v-col cols="12" md="4">
									<p class="font-weight-bold">Información</p>
									<p class="font-weight-light">Plazo de entrega: {{ selectedTaskShow.deadline }}</p>
									<p class="font-weight-light">Prioridad:  {{ selectedTaskShow.priority }}</p>
									<p class="font-weight-light">Total de horas:  {{ selectedTaskShow.totalHoras }}</p>

									<p class="font-weight-bold mt-5">Horas</p>
									<v-btn block>
										<v-icon>mdi-alarm</v-icon> Iniciar
									</v-btn>

									<p class="font-weight-bold mt-5">Funciones</p>
									<v-btn block @click="testCheckList = !testCheckList">
										<v-icon>mdi-plus</v-icon> Checklist
									</v-btn>
								</v-col>
							</v-row>
						</v-container>
					</v-card-text>
					<v-card-actions>
						<v-spacer></v-spacer>
						<v-btn color="darken-1" @click="onCloseShowTask">Cerrar</v-btn>
					</v-card-actions>
				</div>
				<div v-else>
					<v-card-text>
						Llevar
						<v-progress-linear indeterminate class="mb-0"></v-progress-linear>
					</v-card-text>
				</div>
			</v-card>
		</v-dialog>

	</v-container>
</template>

<script>
import draggable from 'vuedraggable'
import CKEditor from '@ckeditor/ckeditor5-vue2';
import ClassicEditor from '@ckeditor/ckeditor5-build-classic';
import moment from 'moment';
import 'moment/locale/es';

export default {
	components: {
		draggable,
		ckeditor: CKEditor.component
	},
	data() {
		return {
			editor: ClassicEditor,
			editDescription: false,
			newDescription: '',
			columns: [
				{
					id: 1,
					name: 'Inicio',
					cards: [
						{
							name: 'Teste Inicial',
							description: "Lorem Ipsum is simply dummy text of the printing and typesetting industry.",
							id: '2',
							actividad: '',
							listaActividad: [
								{
									id: Math.floor((Math.random() * 1000000) + 1),
									text: 'Actividad 1',
									time_create: '2022-08-07T22:06:20+00:00'
								},
								{
									id: Math.floor((Math.random() * 1000000) + 1),
									text: 'Actividad 2',
									time_create: '2022-08-07T22:06:20+00:00'
								}
							],
							totalHoras: '02:35',
							priority: 'medium',
							deadline: '28/11/2019',
							hover: false
						}
					]
				}
			],
			dialogShowTask: false,
			selectedTaskShow: {},
			isAddingCard: false,
			cardCreating: {},
			isAddingColumn: false,
			nameColumn: '',
			fav: false,

			//Testes
			testCheckList: false,
			editorConfig: {
				toolbar: [
					'heading',
					'|',
					'bold',
					'italic',
					'link',
					'bulletedList',
					'numberedList',
					'|',
					'blockQuote',
				],
				placeholder: 'Ingrese su texto aquí'
			}
		}
	},
	methods: {
		removeClass(clase, elemento) {
			let element = document.getElementById(elemento);
			element.classList.remove(clase);
		},
		formattedDate(date) {
			return moment(date).fromNow();
		},
		addActivity() {
			let id = Math.floor((Math.random() * 1000000) + 1)
			this.selectedTaskShow.listaActividad.push({
				id: id,
				text: this.selectedTaskShow.actividad,
				time_create: '2022-08-07T22:06:20+00:00'
			});
			this.selectedTaskShow.actividad = ''
		},
		onShowTask(element) {
			this.$router.push({ path: '/', query: { task: element.id } })
			this.dialogShowTask = true
			setTimeout(() => {
				this.selectedTaskShow = element
			}, 500)
		},
		onCloseShowTask() {
			this.$router.push({ path: '/', query: {} })
			this.dialogShowTask = false
			this.selectedTaskShow = {}
		},
		addCard(column) {
			this.isAddingCard = true
			column.isAddingCard = true
			column.cards.push({
				id: Math.random(),
				name: '',
				description: '',
				actividad: '',
				listaActividad: [],
				hover: false,
				editable: true
			})

			setTimeout(() => {
				this.$refs['inputAddCard' + column.id][0].focus()
			}, 100)
			this.scrollTo('#idColumn' + column.id, 'top')
		},
		scrollTo(element, direction = 'top') {
			setTimeout(() => {
				let container = this.$el.querySelector(element)
				if (direction == 'top') {
					container.scrollTop = container.scrollHeight
				} else if (direction == 'right') {
					container.scrollLeft = container.scrollWidth
				}
			}, 1)
		},
		addColumn() {
			this.isAddingColumn = true
			setTimeout(() => {
				this.$refs.inputAddColumn.focus()
				this.scrollTo('#board', 'right')
			}, 100)
		},
		confirmAddColumn() {
			//si es vcacio cierra y no agrega la columna 
			if (!this.nameColumn || !0 === this.nameColumn.length) {
				this.isAddingColumn = false
				this.nameColumn = ''
				return
			}
			let column = {
				id: Math.floor((Math.random() * 1000000) + 1),
				name: this.nameColumn,
				cards: []
			}
			this.columns.push(column)
			this.isAddingColumn = false
			this.nameColumn = ''
			this.scrollTo('#board', 'right')
		},
		onAddConfirmCard(column) {
			this.isAddingCard = false
			column.isAddingCard = false
			column.cards = column.cards.filter((card) => {
				if (card.editable) {
					if (card.name || !0 === card.name.length) {
						card.editable = false
						return card
					}
				} else {
					return card
				}
			})

			this.scrollTo('#idColumn' + column.id, 'top')
		}
	},
	watch: {
		$route(to) {
			// react to route changes...
			if (!('task' in to.query)) {
				this.onCloseShowTask()
			}
		}
	},
	mounted() {

		if ('task' in this.$route.query) {
			let task = this.$route.query.task
			for (let iColumn = 0; iColumn < this.columns.length; iColumn++) {
				for (let iCard = 0; iCard < this.columns[iColumn].cards.length; iCard++) {
					if (this.columns[iColumn].cards[iCard].id == task) {
						this.onShowTask(this.columns[iColumn].cards[iCard])
						return
					}
				}
			}
		}
	}
}
</script>


<style>
.clickable {
	cursor: pointer;
}
.ck-editor__editable {
	min-height: 75px !important;
}

#board {
	position: absolute;
	top: 0px;
	left: 0px;
	right: 0px;
	box-sizing: border-box;
	padding: 10px;
	align-items: flex-start;
	overflow-x: auto;
	flex-wrap: nowrap
}

.borderText {
	border: 1px solid
}

.minHeight {
	min-height: 350px
}

.ghost,
.sortable-chesen {
	opacity: 0.5;
}

.draggable {
	border: 1px solid
}

.widthColumn {
	width: 300px !important;
	display: flex;
	flex: 0 0 auto;
}

.cardColumm {
	max-height: 100%;
	flex-direction: column;
	display: flex;
	flex: 0 0 auto;
	align-items: stretch !important;
	box-sizing: border-box;
	box-shadow: 1px 1px 3px -2px #000
}

.cardColummHeader {
	flex: 0 0 auto;
	position: relative;
}

.cardColummContent {
	flex: 1 1 auto;
	margin-bottom: 0;
	overflow-y: auto;
	overflow-x: hidden;
	z-index: 1;
	min-height: 0;
}

.draggable {
	min-height: 150px;
	border: 1px solid;
	display: inline
}

.hoverPointer:hover {
	cursor: pointer;
}

.hoverGrab:hover {
	cursor: grab
}

.button {
	margin-top: 35px;
}

.flip-list-move {
	transition: transform 0.5s;
}

.no-move {
	transition: transform 0s;
}

.list-group {
	min-height: 20px;
}

.list-group-item {
	cursor: move;
}

.list-group-item i {
	cursor: pointer;
}



.scrollBar::-webkit-scrollbar-track {
	-webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
	border-radius: 5px;
	background-color: rgba(0, 0, 0, 0.1);
}

.scrollBar::-webkit-scrollbar {
	width: 6px;
	background-color: rgba(0, 0, 0, 0.1);
}

.scrollBar::-webkit-scrollbar-thumb {
	border-radius: 5px;
	-webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, .3);
	background-color: #555;
}

.button-add-card {
	border: 0px;
}

.v-list-item .v-list-item__title,
.v-list-item .v-list-item__subtitle {
	line-height: 1.2;
	font-size: 14px;
}

.v-list-item {
	min-height: 30px;
}
</style>