import arcpy
import os
from dotenv import load_dotenv

load_dotenv(r"C:\Users\cepag\Documents\LuiggibettoneLocal\APIpython\senhasModelbuilder.env")

def create_field_mappings(field_configs):

    field_mappings = arcpy.FieldMappings()

    for config in field_configs:
        field_map = arcpy.FieldMap()

        if config['field_type'] == 'Text' and 'length' in config:
            field_map.addInputField(config['input_layer'], config['input_field'], 0, config['length'] - 1)
        else:
            field_map.addInputField(config['input_layer'], config['input_field'])

        output_field = field_map.outputField
        output_field.name = config['output_field']
        output_field.type = config['field_type']

        if 'nullable' in config:
            output_field.isNullable = config['nullable']

        if 'length' in config:
            output_field.length = config['length']

        field_map.outputField = output_field

        field_mappings.addFieldMap(field_map)

    return field_mappings


if __name__ == "__main__":

    print('Portal Sign In...')

    arcpy.SignInToPortal(
        os.getenv("ARCGIS_PORTAL_URL"),
        os.getenv("ARCGIS_USERNAME"),
        os.getenv("ARCGIS_PASSWORD")
)

    fc_field_maps = os.getenv("FEATURE_SERVICE_ANDARES")
    fc_base_indoor = os.getenv("FEATURE_SERVICE_BASE_INDOOR")

    arcpy.env.overwriteOutput = False
    arcpy.env.preserveGlobalIds = True
    arcpy.env.overwriteOutput = True

    arcpy.management.MakeFeatureLayer(
        in_features=fc_base_indoor,
        out_layer="UCM_Layer",
        where_clause="")

    print('Exporting Field Maps features...')

    arcpy.conversion.ExportFeatures(
        in_features=fc_field_maps,
        out_features=fr"{arcpy.env.scratchGDB}\FM_Layer_FC",
        where_clause="Editado = '1'")

    print('Buffering Field Maps features...')

    arcpy.analysis.PairwiseBuffer(
        in_features=fr"{arcpy.env.scratchGDB}\FM_Layer_FC",
        out_feature_class=fr"{arcpy.env.scratchGDB}\FM_Layer_FC_Buffer", buffer_distance_or_field="1 Meters")

    print('Selecting candidates...')

    ucm_layer_candidates, _, _ = arcpy.management.SelectLayerByLocation(
        in_layer=["UCM_Layer"],
        select_features=fr"{arcpy.env.scratchGDB}\FM_Layer_FC_Buffer")

    print('Exporting candidates...')

    field_configs = [
        {'input_layer': ucm_layer_candidates, 'input_field': 'OBJECTID', 'output_field': 'ucm_oid_original', 'field_type': 'Integer'},
        {'input_layer': ucm_layer_candidates, 'input_field': 'andar', 'output_field': 'ucm_andar', 'field_type': 'SmallInteger'}
    ]

    field_mappings = create_field_mappings(field_configs)

    arcpy.conversion.ExportFeatures(
        in_features=ucm_layer_candidates,
        out_features=fr"{arcpy.env.scratchGDB}\UCM_Layer_Candidates_FC",
        field_mapping=field_mappings)

    print('Performing Spatial Join -> Candidates X Local Field Maps...')

    input_layer1 = fr"{arcpy.env.scratchGDB}\UCM_Layer_Candidates_FC"
    input_layer2 = fr"{arcpy.env.scratchGDB}\FM_Layer_FC"

    field_configs = [
        {'input_layer': input_layer1, 'input_field': 'ucm_oid_original', 'output_field': 'ucm_oid_original', 'field_type': 'Integer'},
        {'input_layer': input_layer1, 'input_field': 'ucm_andar', 'output_field': 'ucm_andar', 'field_type': 'SmallInteger'},
        {'input_layer': input_layer2, 'input_field': 'local', 'output_field': 'local', 'field_type': 'Text', 'length': 10},
        {'input_layer': input_layer2, 'input_field': 'Andar', 'output_field': 'Andar', 'field_type': 'SmallInteger'},
        {'input_layer': input_layer2, 'input_field': 'edificio', 'output_field': 'edificio', 'field_type': 'Text', 'length': 100},
        {'input_layer': input_layer2, 'input_field': 'setor', 'output_field': 'setor', 'field_type': 'Text', 'length': 15},
        {'input_layer': input_layer2, 'input_field': 'departamen', 'output_field': 'departamento', 'field_type': 'Text', 'length': 20},
        {'input_layer': input_layer2, 'input_field': 'ambiente', 'output_field': 'ambiente', 'field_type': 'Text', 'length': 150},
        {'input_layer': input_layer2, 'input_field': 'sigla_amb', 'output_field': 'sigla_amb', 'field_type': 'Text', 'length': 40},
        {'input_layer': input_layer2, 'input_field': 'sub_tp_amb', 'output_field': 'sub_tp_amb', 'field_type': 'Text', 'length': 30},
        {'input_layer': input_layer2, 'input_field': 'cod_edif', 'output_field': 'cod_edif', 'field_type': 'Integer'},
        {'input_layer': input_layer2, 'input_field': 'sigla_edif', 'output_field': 'sigla_edif', 'field_type': 'Text', 'length': 30},
        {'input_layer': input_layer2, 'input_field': 'cod_amb', 'output_field': 'cod_amb', 'field_type': 'Text', 'length': 20},
        {'input_layer': input_layer2, 'input_field': 'tipo_amb', 'output_field': 'tipo_amb', 'field_type': 'SmallInteger'},
        {'input_layer': input_layer2, 'input_field': 'nome_prof', 'output_field': 'nome_prof', 'field_type': 'Text', 'length': 150},
        {'input_layer': input_layer2, 'input_field': 'capacidade', 'output_field': 'capacidade', 'field_type': 'Integer'},
        {'input_layer': input_layer2, 'input_field': 'label', 'output_field': 'label', 'field_type': 'Text', 'length': 80},
    ]

    field_mappings = create_field_mappings(field_configs)

    arcpy.analysis.SpatialJoin(
        target_features=fr"{arcpy.env.scratchGDB}\UCM_Layer_Candidates_FC",
        join_features=fr"{arcpy.env.scratchGDB}\FM_Layer_FC",
        out_feature_class=fr"{arcpy.env.scratchGDB}\UCM_FM_SpatialJoin",
        join_type="KEEP_COMMON",
        field_mapping=field_mappings,
        match_option="CONTAINS",
        match_fields=[["Andar", "ucm_andar"]])

    print('Performing Add Join -> Spatial Join to Candidates...')

    ucm_layer_join = arcpy.management.AddJoin(
        in_layer_or_view=ucm_layer_candidates,
        in_field="OBJECTID",
        join_table=fr"{arcpy.env.scratchGDB}\UCM_FM_SpatialJoin",
        join_field="UCM_OID_Original",
        join_type="KEEP_COMMON",
        join_operation="JOIN_ONE_TO_FIRST")[0]

    print('Calculating fields using join...')

    arcpy.management.CalculateFields(
        in_table=ucm_layer_join,
        expression_type="PYTHON3",
        fields=[
            ["L0Andares.local", "!UCM_FM_SpatialJoin.local!", "", ""],
            ["L0Andares.Andar", "!UCM_FM_SpatialJoin.Andar!", "", ""],
            ["L0Andares.edificio", "!UCM_FM_SpatialJoin.edificio!", "", ""],
            ["L0Andares.setor", "!UCM_FM_SpatialJoin.setor!", "", ""],
            ["L0Andares.departamento", "!UCM_FM_SpatialJoin.departamento!", "", ""],
            ["L0Andares.ambiente", "!UCM_FM_SpatialJoin.ambiente!", "", ""],
            ["L0Andares.sigla_amb", "!UCM_FM_SpatialJoin.sigla_amb!", "", ""],
            ["L0Andares.sub_tp_amb", "!UCM_FM_SpatialJoin.sub_tp_amb!", "", ""],
            ["L0Andares.cod_edif", "!UCM_FM_SpatialJoin.cod_edif!", "", ""],
            ["L0Andares.sigla_edif", "!UCM_FM_SpatialJoin.sigla_edif!", "", ""],
            ["L0Andares.cod_amb", "!UCM_FM_SpatialJoin.cod_amb!", "", ""],
            ["L0Andares.tipo_amb", "!UCM_FM_SpatialJoin.tipo_amb!", "", ""],
            ["L0Andares.nome_prof", "!UCM_FM_SpatialJoin.nome_prof!", "", ""],
            ["L0Andares.capacidade", "!UCM_FM_SpatialJoin.capacidade!", "", ""],
            ["L0Andares.label", "!UCM_FM_SpatialJoin.label!", "", ""]
        ])
