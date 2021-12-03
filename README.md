# Netsuite
Codigo sobre Netsuit
/**
 * @NApiVersion 2.x
 * @NScriptType Suitelet
 * @NModuleScope SameAccount
 * @author Jackie Valencia
 * @description This's a sample SuiteLet script(SuiteScript 2.0) to export data
 *              to Excel file and directly download it in browser
 */
define(['N/file', 'N/encode', 'N/search', 'N/record', 'N/format'],
	/**
	 * @param {file}
	 *            file
	 * @param {format}
	 *            format
	 * @param {record}
	 *            record
	 * @param {redirect}
	 *            redirect
	 * @param {runtime}
	 *            runtime
	 * @param {search}
	 *            search
	 * @param {serverWidget}
	 *            serverWidget
	 */
	function(file, encode, SEARCHMODULE, r, format) {
		/**
		 * Definition of the Suitelet script trigger point.
		 *
		 * @param {Object}
		 *            context
		 * @param {ServerRequest}
		 *            context.request - Encapsulation of the incoming
		 *            request
		 * @param {ServerResponse}
		 *            context.response - Encapsulation of the Suitelet
		 *            response
		 * @Since 2015.2
		 */
		function onRequest(context) {

			function formatDate(testDate) {
				log.debug('testDate: ' + testDate);
				var responseDate = format.format({
					value: testDate,
					type: format.Type.DATE
				});
				log.debug('responseDate: ' + responseDate);
				return responseDate;
			}


			//date = date1.getDate('DD');
			//year = date1.getFullYear('AAAA');
			//month = date1.getMonth('MM');


			if (context.request.method == 'GET') {

				//Lee los parametros de la Pantalla del usuario
				/* Descomentar para depurar
				var Subsidiary = 141;
				var Periodo = 10;*/
				/* Descomentar para depurar*/

				/* Descomentar para Productivo*/
				try{var Subsidiary = context.request.parameters.sub;}catch(ex){var Subsidiary = 1;}
				try{var Periodo = context.request.parameters.period;}catch(ex){var Periodo= 1;}
				/* Descomentar para Productivo*/

				//Termina de Leer los parametros de la Pantalla del usuario  


				//Carga las caracteristicas del Periodo 
				var period = r.load({
					type: r.Type.ACCOUNTING_PERIOD,
					id: Periodo
				});

				//Obtiene Fecha inicial del Periodo
				var startDate = period.getValue({
					fieldId: 'startdate'
				});

				//Obtiene Fecha final del Periodo
				var endDate = period.getValue({
					fieldId: 'enddate'
				});

				//var trandate = period.getFullYear({
				//fieldId: 'trandate'
				//});




				var Content = ''; // Varible que almacena el contenido del Documento

				var fecha = formatDate(endDate); //Da formato a la fecha inicial
				var fechai = formatDate(startDate); //Da formato a la fecha final

				//Crea Varible de Filtros a aplicar a la busqueda
				var filters = [
				
				
                        SEARCHMODULE.createFilter({
                        name: 'trandate',
                        operator: SEARCHMODULE.Operator.WITHIN,
                        values: [fechai,fecha]
                        }),                            
                        SEARCHMODULE.createFilter({
                        name: 'internalid',
                        join: 'Subsidiary',
                        operator: SEARCHMODULE.Operator.ANYOF,
                        values: Subsidiary
                        }),
                        

					/*SEARCHMODULE.createFilter({
						name: 'type',
						operator: SEARCHMODULE.Operator.ANYOF,
						values: ['VendBill', 'VendCred', 'CustPymt'] //'VendPymt', factura de compra, Nota de credito, pago de factura						
					}),*/

					SEARCHMODULE.createFilter({
						name: 'type',
						operator: SEARCHMODULE.Operator.ANYOF,
						values: 'VendBill'
					})
					/*,
											 
					 SEARCHMODULE.createFilter({
					 name: 'mainline',
					 operator: SEARCHMODULE.Operator.IS,
					 values: 'T'
					                        })*/

					/*SEARCHMODULE.createFilter({
                    name: 'account',	
                    operator: SEARCHMODULE.Operator.ANYOF,
                    values: '917'
                     })
                    SEARCHMODULE.createFilter({
                        name: 'item',
                        operator: SEARCHMODULE.Operator.ANYOF,
                        values: [75, 83]
                    }),*/

				];
				//Termina de Crear Varible de Filtros a aplicar a la busqueda

				var settings = [{
					name: 'consolidationtype',
					value: 'NONE'
				}];


				//Crea Varible de Resultados a obtener de la busqueda
				var columnss = [
					//0
					SEARCHMODULE.createColumn({
						'name': 'custentity_rnc',
						'join': 'vendor'
							//1
					}) //RNC p0 var rn vendor
					, SEARCHMODULE.createColumn({
						'name': 'custentitycedula',
						'join': 'vendor'
							//2
					}) //CEULA p1 var rn
					, SEARCHMODULE.createColumn({
						'name': 'custentity_pasaporte',
						'join': 'vendor'
							//3
					}) //PASAPORTE p2 var rn
					, SEARCHMODULE.createColumn({
						'name': 'internalid',
						'join': 'vendorLine'
					}) //Tipo ID
					//4
					, SEARCHMODULE.createColumn({
						'name': 'custbody_tipogasto'
					}) //Tipo Bienes y Servicios comprados Var:BS
					//5
					, SEARCHMODULE.createColumn({
						'name': 'tranid'
					}) //Número comprobante fiscal VAR: NCF
					//6
					, SEARCHMODULE.createColumn({
						'name': 'type'
					}) //Número comprobante fiscal modificado VAR: NCFM
					//7
					, SEARCHMODULE.createColumn({
						'name': 'formulatext',
						'formula': "TO_CHAR ({trandate}, 'YYYYMMDD')"

					}) //Fecha Comprobante (fecha transaccion) VAR: FC
					//8
					, SEARCHMODULE.createColumn({
						'name': 'formulatext',
						'formula': "TO_CHAR ({trandate}, 'YYYYMMDD')",
						'join': 'applyingTransaction'
					}) //Fecha de Pago VAR: FP
					//9
					, SEARCHMODULE.createColumn({
						'name': 'amount'
					}) //Monto 

					//10
					, SEARCHMODULE.createColumn({
						'name': 'taxtotal'
					})
					//ITBIS Facturado VAR:ITF 
					//11
					, SEARCHMODULE.createColumn({
						'name': 'custcol_4601_witaxamount'
					}) //ITBIS Retenido var=IR 
					//12
					, SEARCHMODULE.createColumn({
						'name': 'custcol_4601_witaxcode'
					})
					//ITBIS sujeto a proporcionalidad var= ITP
					//
					//,SEARCHMODULE.createColumn({'name':''})//Tipo de retención en ISR
					//13
					, SEARCHMODULE.createColumn({
						'name': 'custcol_4601_witaxrate'
					}) //Monto retención Renta var:MRT
					//14
					, SEARCHMODULE.createColumn({
						'name': 'taxcode'
					}) //ISR Percibido en compras
					//15
					, SEARCHMODULE.createColumn({
						'name': 'statusref'
					}) //status
					//16
					, SEARCHMODULE.createColumn({
						'name': 'custcol_tipo'
					}) // Biene O SERVICIO 
					//17
					, SEARCHMODULE.createColumn({
						'name': 'transactionnumber'
					}) //no. transaccion (TN)

					//18
					, SEARCHMODULE.createColumn({
						'name': 'custbody_mx_payment_method',
						'join': 'applyingTransaction'
					}) //no. transaccion (TN)

				];
				//Termina de Crear Varible de Resultados a obtener de la busqueda

				//Crea el Modulo de Busqueda guardada y carga los paramet|ros de filtros y columnas
				var s = SEARCHMODULE.create({
					'type': 'transaction',
					'filters': filters,
					'columns': columnss,
					'settings': settings
				}).run(); // Corre la Busqueda

				//Inicia try para leer los resultados
				try {
					s = s.getRange(0, 1000); // establece el rango de resultados
					var total = s.length - 1; //obtiene el index de resultados

					for (var i = 0; i < total; i++) {
						var result = s[i]; // result = al indice de fila de la busqueda guardada



						if (result.getValue(result.columns[5]) !== '' //información en el campo “NCF”
							&&result.getValue(result.columns[16]) !== '' //solo los que tienen bien o servicio
							//&& result.getValue(result.columns[15]) == ['paidInFull','open'] 
							//|| result.getValue(result.columns[15]) === 
							//|| result.getValue(result.columns[15]) !== 'cancelled'//no tomar en  cuenta los cancelados
							//&& result.getValue(result.columns[15]) !== 'rejected'
							//&& result.getValue(result.columns[15]) !== ''

							//&& result.getValue(result.columns[6]) !== 'Journal'
							//&& result.getValue(result.columns[6]) !== 'Check'
						) {

							try {
								var rn = '';
								var ide;
								if (result.getValue(result.columns[0]) !== '') //RNC
								{
									rn = result.getValue(result.columns[0]);
									ide = '1';
								} else if (result.getValue(result.columns[1]) !== '') {
									rn = result.getValue(result.columns[1]);
									ide = '2';
								} else if (result.getValue(result.columns[2]) !== '') {
									rn = result.getValue(result.columns[2]);
									ide = '3';
								} else {
									rn = '';
									ide = '';
								}
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna rnc",
									details: ex.message
								});
								var rn = ''; //cedula, rnc, pasaporte
								var ide = ''; //identificador
							}
							try {
								var BS = result.getValue(result.columns[4]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna BS",
									details: ex.message
								});
								var BS = '';
							} //bienes y servicios

							try {
								var NCF = result.getValue(result.columns[5]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna NCF",
									details: ex.message
								});
								var NCF = '';
							}

							try {
								var NCFM = '';
								if (result.getValue(result.columns[6]) == 'VendCred') //si es nota de credito
									NCFM = result.getValue(result.columns[5]);
								else {
									NCFM = '';
								}
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna NC",
									details: ex.message
								});
								var NCFM = '';
							}

							try {
								var FC = result.getValue(result.columns[7]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna FC",
									details: ex.message
								});
								var FC = '';
							}

							try {
								var FP = result.getValue(result.columns[8]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna FP",
									details: ex.message
								});
								var FP = '';
							}

							/* try {
								var MS = result.getValue(result.columns[9]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna MS",
									details: ex.message
								});
								var MS = '';
							}
							*/


							try
							{
								var ITF = 0;
								if (parseFloat(result.getValue(result.columns[10]) > 0))
								{
									ITF = result.getValue(result.columns[10])
								}else{
									
									ITF = ''
								}
								} catch (ex) {
									log.error({
										title: "Error Leyendo Columna ITF",
										details: ex.message
									});
							}


							try {
								var ITR ='';
								if (result.getValue(result.columns[8]) !== '') //si tiene fecha de pago
								{
									if (
										result.getValue(result.columns[12]) == 6 ||
										result.getValue(result.columns[12]) == 8 ||
										result.getValue(result.columns[12]) == 9 ||
										result.getValue(result.columns[12]) == 17 ||
										result.getValue(result.columns[12]) == 10
									) //impuestos retenidos validos
									{
										ITR = ITR + parseInt(result.getValue(result.columns[11])); //sumatoria de importe de impuestos retenidos	
									} else {
										ITR;
									}
								}
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna ITR",
									details: ex.message
								});
							}



							try {
								var PRO = '';
								if (result.getValue(result.columns[6]) == 'VendCred' || //solo tipo nota de credito 
									result.getValue(result.columns[6]) == 'VendBill' && //solo factura de compra
									result.getValue(result.columns[14]) == '126') { //ITBIS Sujeto a proporcionalidad
									PRO = result.getValue(result.columns[10]);
								} else {
									PRO = '';
								}
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna PRO",
									details: ex.message
								});
								var PRO = '';
							}

							//17 Tipo retencion en ISR
							try {
								var TR = '';

								if (result.getValue(result.columns[12]) == '11') { //Alquileres
									TR = '1'
								} else if (result.getValue(result.columns[12]) == '12') { //Honorarios por servicios     
									TR = '2'
								} else if (result.getValue(result.columns[12]) == '13') { //Otras Rentas 
									TR = '3'
								} else if (result.getValue(result.columns[12]) == '14') { //Otras rentas presuntas 
									TR = '4'
								} else if (result.getValue(result.columns[12]) == '15') { //Intereses pagados a personas jurídicas residentes.
									TR = '5'
								} else if (result.getValue(result.columns[12]) == '16') { //Intereses pagados a personas jurídicas residentes.
									TR = '5'
								} else if (result.getValue(result.columns[12]) == '17') { //Intereses pagados a personas físicas residentes
									TR = '6'
								} else {
									TR = '';
								}
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna TR",
									details: ex.message
								});
								var TR = '';
							}
							//18 Monto retencion Renta
							try {
								var MRT = 0;

								if (result.getValue(result.columns[8]) !== '') //si contiene fecha de pago	

								{ //Servicios * el porcentaje de la retencion
									MRT = (~~parseInt(MFS) / 100) * ~~parseInt(result.getValue(result.columns[9]));
								} else {
									MRT = '';
								}
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna MRT",
									details: ex.message
								});
								var MRT = '0';
							}


							try {
								var STATUS = result.getValue(result.columns[15]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna STATUS",
									details: ex.message
								});
								var STATUS = '';
							}

							try {
								var type = result.getValue(result.columns[6]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna type",
									details: ex.message
								});
								var type = '';
							}

							try {
								var BYS = result.getValue(result.columns[16]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna BYS",
									details: ex.message
								});
								var BYS = '';
							}

							try {
								var TN = result.getValue(result.columns[17]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna TN",
									details: ex.message
								});
								var TN = '';
							}

							try {
								var MFB = 0;

								if (result.getValue(result.columns[16]) == '2') //tipo Bien

								{
									MFB = parseInt(MFB) + parseInt(result.getValue(result.columns[9]));
								} else {
									MFB = '';
								}

							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna MFB",
									details: ex.message
								});
								var MFB = '0';
							}

							try {
								var MFS = 0;

								if (result.getValue(result.columns[16]) == '1') //tipo Sevicio

								{
									MFS = parseInt(MFS) + parseInt(result.getValue(result.columns[9]))
								} else {
									MFS = '';
								}
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna MFS",
									details: ex.message
								});
								var MFS = '0';
							}

							try {
								var TP = result.getValue(result.columns[18]);
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna TP",
									details: ex.message
								});
								var TP = '';
							}


							try {
								var MTF = 0; // Total monto facturado
								MTF = MFB + MFS; //sumatoria de bienes y servicios                           
							} catch (ex) {
								log.error({
									title: "Error Leyendo Columna MTF",
									details: ex.message
								});
							}
							//Tipo de pago 18    
							/*try {
									var TP = result.getValue(result.columns[18]);
								} catch (ex) {
									log.error({
										title: "Error Leyendo Columna TP",
										details: ex.message
									});
									var TP = '';
*/



							Content += rn + '|' + ide + '|' + BS + '|' + NCF + '|' + NCFM + '|' + FC + '|' + FP + '|' + MFS + '|' + MFB + '|' + MTF + '|' + ITF + '|' + ITR + '|' + PRO +
								'|' + '|' + '|' + '|' + TR + '|' + MRT + '|' + '|' + '|' + '|' + TP +'\n';
						} //fin if		
					} //fin del for
				}

				//En caso de error llena el contenido con un mensaje de error
				catch (ex) {
					log.error({
						title: "Error al Leer los resultados de la busqueda",
						details: ex.message
					});
					var Content = 'Error al Leer la Busqueda, Revisa el Log de Errores';
				}


				// Traspasa el contenido a una variable y lo codifica
				var contenido = encode.convert({
					string: Content,
					inputEncoding: encode.Encoding.UTF_8,
					outputEncoding: encode.Encoding.UTF_8
				});
				// Termina de Traspasar el contenido a una variable y lo codifica

				// Establece propuedades del documento 
				var fileRequest = {
					name: 'Reporte 606' + '.txt', //Nombre y extension del Documento
					fileType: file.Type.PLAINTEXT, //Tipo de Documento
					contents: contenido, //Contenido
					folder: 332 //folder en caso de aplicar
				}

				// almacena el Documento en el repositorio
				try {
					var resultingFile = file.create(fileRequest);
					var fileId = resultingFile.save();
				} catch (ex) {
					document.write(ex);
					debugger;
				} // almacena el Documento en el repositorio


				//descarga el documento
				context.response.writeFile({
					file: resultingFile
				});
			}

		}
		return {
			onRequest: onRequest
		};

	});
